# amr-claude-plugins

Personal Claude Code plugins, one subdirectory per plugin, installable independently from this one repo via Claude Code's plugin marketplace mechanism.

## Install on any machine

```
/plugin marketplace add Relatora/amr-claude-plugins
/plugin install release-notes
/plugin install detailed-commit
```

Each plugin installs and updates independently — you don't need all of them.

## What's here

| Plugin | What it does |
|---|---|
| [release-notes](release-notes/) | Generates a release notes markdown file from a repo's pending changes, plus a bullet list for an issue tracker. |
| [detailed-commit](detailed-commit/) | Drafts a detailed git commit message (subject + bullets + optional `Issue #NN` footer), asking for the issue number if it's not given. |

## Adding a new plugin to this repo

1. Author and test the skill's `SKILL.md` wherever's convenient (a project's own `.claude/skills/` while iterating works fine).
2. Add a new top-level directory here named after the plugin:
   ```
   <plugin-name>/
   ├── .claude-plugin/plugin.json   # { "name", "description", "version": "0.1.0" }
   ├── skills/<plugin-name>/SKILL.md
   └── README.md
   ```
3. Add an entry for it to the root `marketplace.json`'s `plugins` array:
   ```json
   { "name": "<plugin-name>", "source": { "type": "github", "repo": "Relatora/amr-claude-plugins", "path": "<plugin-name>" } }
   ```
4. Commit and push.
5. `/plugin install <plugin-name>` on any machine that already has this marketplace added (run `/plugin marketplace update` first if it doesn't pick up the new entry automatically).

Bump the `version` field in a plugin's `plugin.json` whenever you edit that plugin, so machines that already installed it can pick up the update.
