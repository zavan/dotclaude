---
name: stacked-pr
description: Split work into a stack of dependent pull requests with GitHub's `gh stack` extension, and drive that stack from an agent — planning layers before writing code, propagating fixes upward, syncing after merges, and merging bottom-up. Use when the user types /stacked-pr, asks for stacked PRs, a PR stack, layered or incremental PRs, or when a task you are about to implement is large enough that one PR would be unreviewable.
---

# Stacked pull requests

A stack is a chain of PRs in one repository. The bottom PR targets the trunk; every PR above it
targets the branch below. Each layer is one discrete, reviewable change, and reviewers see only
that layer's diff. GitHub re-targets the remaining PRs as layers merge.

The rule that governs every decision below: **if code in one layer depends on code in another, the
dependency must live in the same layer or a lower one.**

## Step 1: decide whether to stack

Stack when the work has a dependency order that a reviewer would otherwise have to untangle:
schema before endpoints, endpoints before UI, refactor before the feature that needs it. Stack when
you are generating a volume of code that would land as one large diff — the review bottleneck is the
reason this feature exists.

Do not stack when:

- The change is one coherent thing. A three-layer stack of 40 lines each is overhead, not clarity.
- The pieces are independent. Independent work wants separate PRs off trunk, not a chain — a stack
  forces a serial merge order you do not need.
- The branches would live in different repositories or in a fork. Cross-fork stacks do not exist.

Creating PRs is outward-facing: it notifies reviewers and consumes their attention. Get the user's
go-ahead before `gh stack submit`, and describe the layer breakdown you intend to push. Building the
branches locally needs no such approval.

## Step 2: design the layers before writing code

Do this first, always. Restructuring a stack later is the expensive operation; planning it is
cheap. Write the plan out and check it against four tests:

1. **Coherent** — each layer is one change you can name in a short PR title without "and".
2. **Ordered** — nothing in a layer references anything above it.
3. **Reviewable alone** — a reviewer who reads only this diff can judge it.
4. **Green alone** — the layer builds and its tests pass at that point in the chain. CI runs on
   every PR in the stack, and a layer that only compiles once the layer above lands will sit red
   and block the merge.

Test 4 is the one that bites: it decides where tests go. Tests for a layer belong *in* that layer
whenever they can pass there. A trailing "tests" layer is acceptable only for tests that genuinely
cannot exist until the whole feature does, such as end-to-end tests.

A typical shape:

| Layer | Contents |
|---|---|
| 1 | Data model, migration |
| 2 | CRUD endpoints + their unit tests |
| 3 | Auth middleware and guards + their tests |
| 4 | End-to-end tests |

If a layer's diff grows past what reads quickly, split it before you push rather than after.

## Step 3: preflight

Check the environment once, before touching branches:

```sh
gh --version                       # needs 2.90.0 or later; git needs 2.20+
gh auth status
gh extension list | grep gh-stack  # else: gh extension install github/gh-stack
```

If any `gh stack` command exits **9**, stacked pull requests are not enabled for that repository.
Stop and tell the user — this is a repository setting, not something to work around by hand-building
branches with `--base`.

## Step 4: build the stack

```sh
gh stack init LAYER-1-BRANCH        # -b, --base <branch> to target a non-default trunk
# ...write code...
git add . && git commit -m "..."

gh stack add LAYER-2-BRANCH         # new branch on top of the current one
# ...write code...
git add . && git commit -m "..."

gh stack push                       # push branches only
gh stack submit                     # push, then create/update the PRs and link the stack
```

`gh stack add -Am "MESSAGE"` stages, commits, and branches in one call. Its behaviour is
conditional: if the current branch has no commits yet the change lands there, otherwise it creates a
new branch. That conditional makes it a poor fit for scripted use — prefer the explicit
`git commit` then `gh stack add BRANCH` sequence, where you always know which branch you are on.

Useful flags on submit: `--auto` to enable auto-merge, `--open` to open the PRs in a browser (skip
this in an agent context), `--remote <name>` for a non-`origin` remote.

Give each PR a description that names its layer's scope and what it assumes from below. Reviewers
of layer 3 need to know that the middleware contract came from layer 2.

## Step 5: read the stack before acting on it

Never guess the current position. `gh stack view --json` returns the branches, PR numbers, and
statuses in machine-readable form; `gh stack view -s` is the short human form. Read it before any
rebase, merge, or navigation decision, and again after anything that could have changed the remote.

Navigation: `gh stack checkout BRANCH` (also accepts a stack number, PR number, or URL),
`gh stack up`, `gh stack down`, `gh stack top`, `gh stack bottom`, `gh stack trunk`. Use `checkout`
with an explicit branch name in scripted work; `up`/`down` are relative and easy to lose track of.
`gh stack switch` is interactive — see step 8.

## Step 6: change a lower layer

This is the operation you will perform most, because review feedback almost always lands below where
you are working. Fix the code where it belongs — never patch a lower layer's bug in an upper layer.

```sh
gh stack checkout LOWER-BRANCH
# ...fix...
git add . && git commit -m "..."
gh stack rebase --upstack     # cascade the change into every layer above
gh stack push
gh stack top                  # or checkout back to wherever you were
```

`gh stack rebase` with no flag rebases the whole stack from trunk upward. `--downstack` covers the
lowest layer up to the current branch, `--upstack` covers the current branch to the top. Other
flags: `--no-trunk`, `--remote <name>`, `--committer-date-is-author-date`.

After the cascade, verify the layers above still work. A rebase resolves text, not meaning — if you
changed a function signature in layer 1, layer 3 can rebase cleanly and still be broken.

## Step 7: conflicts and error handling

A cascading rebase stops at the first conflict. Resolve, stage, continue:

```sh
# resolve the <<<<<<< markers
git add .
gh stack rebase --continue
gh stack rebase --abort        # or: restart from the pre-rebase state
```

Branch on exit codes rather than on parsed output:

| Code | Meaning | Response |
|---|---|---|
| 0 | Success | continue |
| 1 | Generic error | read stderr |
| 2 | Not in a stack, or stack not found | `gh stack view` to reorient |
| 3 | Rebase conflict | resolve, `git add`, `--continue` |
| 4 | GitHub API failure | retry once, then report |
| 5 | Invalid arguments or flags | fix the call |
| 6 | Disambiguation required | branch is in several stacks; name the stack |
| 7 | Rebase already in progress | finish or abort it first |
| 8 | Stack locked by another process | another `gh stack` is running; wait |
| 9 | Stacked PRs not enabled for the repository | stop, tell the user |
| 10 | Modify session interrupted | `gh stack modify --abort` |

If `gh stack sync` halts on conflicts the branches are left partially updated. Recover with
`gh stack rebase`, then `gh stack push`.

Resolve mechanical conflicts yourself — the same rename on both sides, an import list, a changelog.
Stop and ask when the resolution is a judgment call about which behaviour is correct, especially in
a lower layer where a wrong guess propagates into every layer above.

## Step 8: what not to run unattended

`gh stack modify` opens an interactive TUI for restructuring — drop (`x`), fold down (`d`), fold up
(`u`), insert (`i`/`I`), rename (`r`), reorder (Shift+↑/↓), undo (`z`), save (Ctrl/Cmd+S). An agent
cannot drive that reliably. Do not launch it; describe the restructuring you want and hand the user
the keystrokes, or rebuild the affected layers with ordinary commands. `--continue` and `--abort`
are the only non-interactive entry points, and they exist to recover a session someone else started.

`gh stack switch` is likewise interactive — use `gh stack checkout BRANCH`.

Also avoid:

- **Raw `git rebase` or `git push --force` on a stack branch.** The extension maintains tracking
  state and PR bases; hand-rebasing desynchronizes both. Use `gh stack rebase` and `gh stack push`.
- **The web "Rebase stack" button** when the repository requires signed commits — server-side
  rebases produce unsigned commits. `gh stack rebase` respects local signing config.
- **Closing a PR in the middle of a stack.** It blocks every layer above from merging, and the fix
  is to dissolve and recreate the stack. Empty a layer instead, or restructure deliberately.

## Step 9: merge and sync

Merge bottom-up. `gh stack merge` takes one or more PRs and accepts `--merge`, `--squash`,
`--rebase`, or `--merge-method <method>`, plus `-y` to skip the prompt. GitHub re-targets the next
layer automatically as each one lands. Every PR in the stack and below it must have its required
reviews and passing checks, and the stack must have linear history — if a merge is refused for
non-linearity, run `gh stack rebase` then `gh stack push`.

A mid-stack merge failure leaves the merged layers on the base branch and the rest open. Fix the
failing PR and re-run the merge.

After anything lands — including a merge someone else performed:

```sh
gh stack sync --prune
```

This fetches, fast-forwards trunk, rebases the remaining branches, pushes, and syncs PR state,
dropping merged branches. When local and remote have diverged it asks whether to take the remote as
the source of truth, delete the stack on GitHub, or cancel; that is a decision for the user, so
surface the divergence rather than picking for them.

`gh stack unstack` dissolves a stack: open, draft, and closed PRs are unlinked, while merged and
queued PRs stay. `--local` removes local tracking only. `gh stack link` does the inverse — links
existing PRs into a stack on GitHub without local tracking.

## Reference

`references/commands.md` has the full command and flag list.
