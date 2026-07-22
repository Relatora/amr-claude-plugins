# v1.1.0 Release Notes

## New: detailed-commit auto-pushes after committing

The `detailed-commit` skill no longer stops after a clean commit — it now pushes automatically.

- Step 5 added to [detailed-commit/skills/detailed-commit/SKILL.md](../../detailed-commit/skills/detailed-commit/SKILL.md): runs `git push`, or `git push -u origin <branch>` if the branch has no upstream yet. Never force-pushes unless the user explicitly asks.

## detailed-commit reliability fixes

- Step 2 now asks the user (via `AskUserQuestion`) before staging everything, whenever nothing is already staged: "Stage everything and commit it all" vs "I'll stage what I want myself." Declining, or a non-interactive session, defaults to staging everything — the skill no longer silently skips files it should have committed.
- Step 4 now verifies the actual commit message content, not just tree cleanliness: after committing, it runs `git log -1 --format=%B` and checks the result against the step-3 draft (bullets present, `Issue #<N>` footer landed). `git status` alone only proves the working tree is clean — it says nothing about whether the heredoc dropped a line. If the footer or a bullet is missing, the skill now fixes it immediately with `--amend` (carved out as the one case amending is expected, since nothing's pushed yet at that point) instead of leaving a malformed message in history.

## release-notes: version-conflict guard before augmenting

Previously, if `docs/Releases/RELEASE_NOTES_v<version>.md` already existed at the manifest's current version, the skill always assumed it was still in-progress and augmented it — even if that version had actually already shipped, which would have silently buried new commits inside an already-released doc.

- Step 2 ([release-notes/skills/release-notes/SKILL.md](../../release-notes/skills/release-notes/SKILL.md)) now checks for a release marker (a matching git tag, or a `vX.Y.Z: <summary>` commit) at or after the existing doc's last-touched point before deciding to augment.
- Marker found → treated as a new release; skill goes straight to writing a new doc at the next version instead of touching the old one.
- No marker and genuinely ambiguous → skill now asks via `AskUserQuestion` whether to augment the existing doc, start a new doc at a specific next version, or take a custom version — instead of guessing.
- Step 6's augment flow now explicitly only runs once step 2 has confirmed the target doc is still in-progress.

## Misc

- Both plugins' manifests ([detailed-commit/.claude-plugin/plugin.json](../../detailed-commit/.claude-plugin/plugin.json), [release-notes/.claude-plugin/plugin.json](../../release-notes/.claude-plugin/plugin.json)) now include `author` (name + GitHub URL) — `claude plugin` tag validation was flagging both as missing this.

---

### Upgrade notes

No breaking changes. `release-notes/.claude-plugin/plugin.json` is now bumped to `1.1.0`. `detailed-commit/.claude-plugin/plugin.json` still reads `"version": "1.0.0"` — bump it to `1.1.0` manually before tagging this release.
