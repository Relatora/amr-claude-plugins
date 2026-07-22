# v1.0.0 Release Notes

## New: repo scaffolded, two plugins shipped

First stable release of `amr-claude-plugins` — a Claude Code plugin marketplace hosting personal skills as independently-installable plugins from one repo.

- New root [.claude-plugin/marketplace.json](../../.claude-plugin/marketplace.json) — lists both plugins below, each installable independently via `/plugin install <name>` after one `/plugin marketplace add Relatora/amr-claude-plugins`.
- **release-notes** plugin ([release-notes/](../../release-notes/)): generates a `docs/Releases/RELEASE_NOTES_v<version>.md` file from a repo's pending/uncommitted changes and unreleased commits, plus a plain bullet list for pasting into an issue tracker. Also merges new commits into an already-written, not-yet-released doc, and cross-links sibling repos in a multi-repo suite. Migrated verbatim from a loose personal skill at `~/.claude/skills/release-notes/SKILL.md`.
- **detailed-commit** plugin ([detailed-commit/](../../detailed-commit/)): drafts a git commit message with a subject line, a bullet list of concrete changes, and an optional `Issue #NN` footer — prompting for the issue number (with an explicit "no issue" option) when one isn't given or inferable. Migrated from a project-local skill originally scoped to a single repo (`agg-client`), generalized so it detects *any* repo's own existing commit convention first and only falls back to the default subject+bullets+footer shape when a repo has no clear convention of its own.

## Marketplace schema fixes

Two schema mistakes were made and corrected while first registering the marketplace against a real Claude Code install (`claude plugin marketplace add`), found by comparing against an already-installed marketplace's cached manifest rather than guessing:

- `marketplace.json` must live at `.claude-plugin/marketplace.json` at the repo root — a bare root-level `marketplace.json` is not discovered.
- The manifest needs top-level `name`, `description`, and `owner` fields, and each plugin entry's `source` must be a path string relative to the marketplace file (`./release-notes`, `./detailed-commit`) rather than a `{type, repo, path}` object — same-repo plugins don't need a github source object at all.

## Misc

- Removed the two now-superseded loose copies once the plugin install was confirmed working: the project-local `detailed-commit` skill in `agg-client`'s `.claude/skills/`, and the loose personal `release-notes` skill at `~/.claude/skills/release-notes/`. Each plugin now has exactly one source of truth.

---

### Upgrade notes

No breaking changes — this is the first tagged release. Anyone who already ran `/plugin marketplace add` + `/plugin install` against a pre-1.0.0 commit should `/plugin update <name>` (or reinstall) to pick up the version bump.
