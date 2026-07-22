# detailed-commit

Claude Code plugin. Drafts and creates a git commit message that actually documents what changed — subject line, bullet list of concrete changes, optional `Issue #NN` footer — instead of a terse one-liner. Prompts for an issue number if one isn't given or inferable from context, with an explicit "no issue" option.

See [skills/detailed-commit/SKILL.md](skills/detailed-commit/SKILL.md) for the full behavior.

Trigger phrases: "detailed commit", "write a detailed commit message", "commit this for issue #NN", or `/detailed-commit`.
