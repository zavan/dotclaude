# `gh stack` command reference

Install: `gh extension install github/gh-stack`. Requires GitHub CLI 2.90.0+ and Git 2.20+.
Optional companion for Copilot CLI: `gh skill install github/gh-stack`.

## Commands

| Command | Purpose | Flags |
|---|---|---|
| `gh stack init` | Initialize a new stack in the current repository | `-b, --base <branch>` |
| `gh stack add` | Add a new branch on top of the current stack | `-A, --all`, `-u, --update`, `-m, --message <string>` |
| `gh stack view` | View the current stack | `-s, --short`, `--json` |
| `gh stack checkout` | Check out a stack by stack number, pull request number, URL, or branch | |
| `gh stack modify` | Interactively restructure the current stack | `--continue`, `--abort` |
| `gh stack unstack` | Remove a stack from local tracking and unstack it on GitHub | `--local` |
| `gh stack submit` | Push branches, then create or update pull requests and stack | `--auto`, `--open`, `--remote <name>` |
| `gh stack sync` | Fetch, rebase, push, and sync pull request state in one command | `--remote <name>`, `--prune` |
| `gh stack rebase` | Pull from remote and run cascading rebase across the stack | `--downstack`, `--upstack`, `--no-trunk`, `--continue`, `--abort`, `--remote <name>`, `--committer-date-is-author-date` |
| `gh stack push` | Push active branches in the current stack to the remote | `--remote <name>` |
| `gh stack link` | Link pull requests into a stack on GitHub without local tracking | `--base <branch>`, `--open`, `--remote <name>` |
| `gh stack merge` | Merge one or more stacked pull requests at once | `--merge-method <method>`, `--merge`, `--squash`, `--rebase`, `-y, --yes` |
| `gh stack switch` | Interactively switch to another branch in the stack | |
| `gh stack up` | Move up toward the top of the stack, away from the trunk | |
| `gh stack down` | Move down toward the bottom of the stack, toward the trunk | |
| `gh stack top` | Jump to the top of the stack | |
| `gh stack bottom` | Jump to the bottom of the stack | |
| `gh stack trunk` | Jump to the trunk branch | |
| `gh stack alias` | Create a short command alias | `--remove` |
| `gh stack feedback` | Share feedback about the gh stack extension | |

Interactive, not agent-drivable: `gh stack modify` (except `--continue`/`--abort`), `gh stack switch`.

## Exit codes

| Code | Meaning |
|---|---|
| 0 | Success |
| 1 | Generic error |
| 2 | Not in a stack, or stack not found |
| 3 | Rebase conflict |
| 4 | GitHub API failure |
| 5 | Invalid arguments or flags |
| 6 | Disambiguation required (branch in multiple stacks) |
| 7 | Rebase already in progress |
| 8 | Stack locked by another process |
| 9 | Stacked pull requests not enabled for repository |
| 10 | Modify session interrupted; recovery required |

## `gh stack modify` keys

| Action | Key |
|---|---|
| Drop branch | `x` |
| Fold into branch below | `d` |
| Fold into branch above | `u` |
| Insert branch | `i` / `I` |
| Rename branch | `r` |
| Reorder | Shift+↑ / Shift+↓ |
| Undo action | `z` |
| Save | Ctrl/Cmd+S |

Prerequisites for a modify session: an active stack checked out, a clean working tree, no rebase in
progress, no PR queued to merge, and linear commit history.

## Platform limits

- All branches in a stack must live in the same repository. Cross-fork stacks are unsupported.
- Not available in GitHub Desktop. Available in GitHub CLI, the web UI, GitHub Mobile, and the
  REST/GraphQL APIs.
- Supports merge commit, squash, and rebase merge methods; works with GitHub Actions CI and merge
  queues.
- Branch protection rules, including CODEOWNER approvals, are enforced on every PR in the stack.
- Server-side rebases (the web "Rebase stack" button) produce unsigned commits.
- A PR ejected from a merge queue ejects every PR above it too.

## Sources

- https://docs.github.com/en/pull-requests/get-started/about-stacked-prs
- https://docs.github.com/en/pull-requests/get-started/stacked-prs-quickstart
- https://docs.github.com/en/pull-requests/how-tos/create-pull-requests/managing-stacked-pull-requests
- https://docs.github.com/en/pull-requests/reference/stacked-prs-cli-commands
- https://docs.github.com/en/pull-requests/how-tos/merge-and-close-pull-requests/troubleshooting-stacked-pull-requests
- https://docs.github.com/en/copilot/tutorials/stack-ai-generated-code-in-pull-requests
