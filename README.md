# dotclaude

My [Claude Code](https://claude.com/claude-code) configuration: user settings and personal skills.

## Layout

```
settings.json      → ~/.claude/settings.json
skills/            → ~/.claude/skills/
```

## Install

```sh
git clone git@github.com:zavan/dotclaude.git ~/projects/dotclaude
ln -s ~/projects/dotclaude/settings.json ~/.claude/settings.json
ln -s ~/projects/dotclaude/skills/asd ~/.claude/skills/asd
```

Symlink each skill individually rather than the whole `skills/` directory, so skills installed by
other means can live alongside these.

## Skills

### `/asd` — ASD-STE100 Simplified Technical English

Rewrites prose into the controlled English used for aerospace and defense maintenance
documentation: approved vocabulary, one word per meaning, active voice, capped sentence length, no
`-ing` nouns, warnings before the steps they apply to.

Two modes:

- **`/asd` alone** restates the previous reply in STE, with the content unchanged.
- **`/asd <request>`** reads the request to find the target — the next reply, a README section,
  docstrings, UI warning strings — and applies STE there only.

The approved ~900-word dictionary ships with the ASD-STE100 specification and is not reproduced
here. The skill works from the rule sections plus a table of high-frequency substitutions, and
flags uncertainty instead of claiming compliance. Good STE-style prose; a formally controlled
deliverable still needs the real dictionary.

## Plugins

Enabled plugins and their marketplaces are declared in `settings.json` under `enabledPlugins` and
`extraKnownMarketplaces`. Claude Code installs them on first run; the resolved install paths and
version pins are machine state and stay out of this repo.

## What is not here

`~/.claude` also holds session transcripts, shell history, per-project memory files, plans, tasks,
and plugin caches. That is local state — machine-specific, or private, or both — and it is
deliberately excluded. See `.gitignore`.
