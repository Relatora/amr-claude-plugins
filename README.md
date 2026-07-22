# amr-claude-plugins

Personal Claude Code plugins, one subdirectory per plugin, installable independently from this one repo via Claude Code's plugin marketplace mechanism.

## Install on a different machine

Prerequisite: Claude Code installed and logged in — no `gh` auth, no clone, no manual file copying needed. This works the same whether you run it from an interactive Claude Code session (slash commands) or from a shell (the `claude plugin` CLI) — pick whichever you're in.

**1. Add this repo as a marketplace** (one-time, registers where to find the plugins below — does not install anything yet):

```text
/plugin marketplace add Relatora/amr-claude-plugins
```
or from a shell:
```bash
claude plugin marketplace add Relatora/amr-claude-plugins
```

**2. Install whichever plugin(s) you want** (repeat per plugin — each installs and updates independently, you don't need both):

```text
/plugin install release-notes@amr-claude-plugins
/plugin install detailed-commit@amr-claude-plugins
```
or:
```bash
claude plugin install release-notes@amr-claude-plugins
claude plugin install detailed-commit@amr-claude-plugins
```

The `@amr-claude-plugins` suffix pins which marketplace to install from — safe to omit if you don't have another marketplace with a same-named plugin, but harmless to always include.

**3. Verify:**

```text
/plugin list
```
or `claude plugin list` — look for `release-notes@amr-claude-plugins` / `detailed-commit@amr-claude-plugins` with `Status: ✔ enabled`.

**4. Use it** — trigger phrases are in each plugin's own README ([release-notes](release-notes/README.md), [detailed-commit](detailed-commit/README.md)), e.g. say "write a detailed commit" or invoke `/detailed-commit` directly.

**Staying up to date:** when a plugin here gets a new version, `claude plugin update <name>` (or `/plugin update <name>`) picks it up — a Claude Code restart is required to apply it.

## What's here

| Plugin | What it does |
|---|---|
| [release-notes](release-notes/) | Generates a release notes markdown file from a repo's pending changes, plus a bullet list for an issue tracker. |
| [detailed-commit](detailed-commit/) | Drafts a detailed git commit message (subject + bullets + optional `Issue #NN` footer), asking for the issue number if it's not given. |

## Adding a new plugin to this repo

1. Author and test the skill's `SKILL.md` wherever's convenient (a project's own `.claude/skills/` while iterating works fine).
2. Add a new top-level directory here named after the plugin:
   ```text
   <plugin-name>/
   ├── .claude-plugin/plugin.json   # { "name", "description", "version": "0.1.0" }
   ├── skills/<plugin-name>/SKILL.md
   └── README.md
   ```
3. Add an entry for it to `.claude-plugin/marketplace.json`'s `plugins` array — `source` is a path relative to the marketplace file, same repo so just `./<plugin-name>`:
   ```json
   { "name": "<plugin-name>", "description": "<one-liner>", "source": "./<plugin-name>", "category": "productivity" }
   ```
4. Commit and push.
5. `/plugin install <plugin-name>` on any machine that already has this marketplace added (run `/plugin marketplace update` first if it doesn't pick up the new entry automatically).

Bump the `version` field in a plugin's `plugin.json` whenever you edit that plugin, so machines that already installed it can pick up the update.

## Releases

- [v1.0.0](docs/Releases/RELEASE_NOTES_v1.0.0.md) — first stable release: both plugins, marketplace schema fixes, old loose skill copies retired.
