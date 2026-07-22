# release-notes

Claude Code plugin. Generates a `docs/Releases/RELEASE_NOTES_v<version>.md` file from a repo's pending/uncommitted changes and unreleased commits, plus a plain bullet list for pasting into an issue tracker. Also merges new commits into an already-written, not-yet-released doc instead of duplicating it, and cross-links sibling repos in a multi-repo suite (e.g. a client + server pair).

See [skills/release-notes/SKILL.md](skills/release-notes/SKILL.md) for the full behavior.

Trigger phrases: "create a release doc/md", "release notes for this commit/branch", "changelog entry", "update the release notes with my new commits", or `/release-notes`.
