---
name: release-notes
description: Generate a release notes markdown file for a repo from its pending/uncommitted changes (and unreleased commits), plus a plain bullet list for pasting into an issue tracker. Also augments an already-written, not-yet-released doc with commits made after it was created, merging into existing sections instead of duplicating. Handles multi-repo suites (e.g. a client+server pair) by cross-linking each repo's doc. Use when the user says "create a release doc/md", "release notes for this commit/branch", "changelog entry", "update the release notes with my new commits", or invokes /release-notes.
---

# release-notes

Produce two things from a repo's pending changes:
1. A committed markdown file at **`docs/Releases/RELEASE_NOTES_v<version>.md`**.
2. A plain bullet list in the chat response (no file) for the user to paste into an issue/ticket.

## 1. Gather the real changes — never fabricate

- `git status` + `git diff` for uncommitted work; `git log --oneline` and `git diff <last-release-tag-or-commit>...HEAD` for already-committed-but-unreleased work. Use whichever combination covers everything since the last release marker.
- Find the last release marker by convention, not guesswork: look for existing `docs/Releases/RELEASE_NOTES_v*.md` files, or commit messages shaped `vX.Y.Z: <summary>` in `git log`. Whichever exists in this repo, follow it.
- Read the actual new/changed files (`Read`, not just the diff hunk) when a diff line references something non-obvious — you need to describe *what* changed, not just quote the patch.
- If a change is a dependency bump, name the package and the old→new version, don't editorialize.
- Every number, filename, and version string in the doc must come from something you actually read this session. If unsure, say so instead of rounding/guessing.

## 2. Determine the next version

- Read the manifest the repo actually uses (`package.json` version, or equivalent — `pyproject.toml`, `Cargo.toml`, `*.csproj`, etc.).
- If that manifest hasn't been bumped yet for this release, pick the next version by the repo's own existing scheme (semver patch/minor per its own history — check the last few `vX.Y.Z:` commit messages for the pattern, e.g. one repo may patch-bump per release, another minor-bump). Each repo's version sequence is independent — never force sibling repos (client/server) to share a version number.
- Flag in your chat response (not in the doc) if the manifest still needs a manual bump before tagging — don't silently edit the version yourself unless asked.
- **If a doc already exists at `docs/Releases/RELEASE_NOTES_v<manifest-version>.md`** (manifest version == an existing doc's version), don't assume that doc is still in-progress. Check for a release marker at or after the doc's last-touched point (see step 6.3 for how to find that point): a git tag matching that version, or a commit message shaped `vX.Y.Z: <summary>` for that same version. That marker means the release already shipped — new commits belong to the *next* version, not that doc.
  - Marker found → this is a new release. Skip straight to writing a new doc at the next version (step 3), don't augment.
  - No marker, genuinely ambiguous → **ask the user** via AskUserQuestion before writing anything: augment the existing `vX.Y.Z` doc (still in progress), start a new `vX.Y.Z+1` doc (already shipped, this is the next release — name the specific next version you'd use as the option label), or let them type a specific version. Never silently pick one when it's unclear — guessing wrong means either polluting an already-shipped release's notes or silently dropping new work into a doc no one will read as current.

## 3. Write the doc

Location: `docs/Releases/RELEASE_NOTES_v<version>.md` at the repo root. Create `docs/Releases/` if it doesn't exist. Never place it at repo root or anywhere else — this is the fixed convention across projects now.

Structure (omit sections that don't apply, don't force empty ones):

```markdown
# v<version> Release Notes

## New: <headline feature name>

<1-2 sentence description of the feature/capability. Cross-link the sibling repo's
release doc here if this feature spans repos — see step 4.>

- <specific sub-point with file link: [file.ext](relative/path)>
- ...

## <Area> improvements

- <change, one line, specific>

## Misc

- <dependency bump / small fix / etc.>

---

### Upgrade notes
<Breaking changes, migrations, new required env vars/config, or "No breaking changes."
If this pairs with a sibling repo's release, say so and note deploy order/together.>
```

Use relative markdown links (`[file.jsx](src/components/Foo.jsx)`) for every file called out, same as normal code-reference style.

## 4. Multi-repo suites — cross-link, don't duplicate

If the working repo has sibling repos (check the parent directory for other git repos touched by the same feature — e.g. a `-client` and `-server` pair, or a monorepo's separate packages), and the change spans both:

- Write one release doc per repo, each under that repo's own `docs/Releases/`.
- Each doc's "New" section references the other repo's doc by name/version (e.g. "see agg-client `docs/Releases/RELEASE_NOTES_v0.1.29.md`"), not by full duplication of its content.
- Note in "Upgrade notes" that the two must deploy together if one depends on the other (e.g. new frontend routes calling new backend endpoints).
- Investigate the sibling repo's actual git diff/log the same way as step 1 — don't infer its contents from the current repo's diff alone.

## 5. Also return bullet points in chat (separate from the file)

After writing the file(s), give the user a plain bullet list per repo, terse, no headers, no markdown links needed — meant for copy-paste into an issue/PR description. This is always a chat response, never written to a file unless asked.

## 6. Augmenting an existing, not-yet-released doc with new commits

When the user made one or more commits *after* a `docs/Releases/RELEASE_NOTES_v<version>.md` for the in-progress version was already written, and wants it updated — merge in, don't create a second file and don't regenerate the whole doc from scratch. This whole section only applies once step 2 has confirmed the target doc is still in-progress (no release marker found, or the user picked "augment") — never augment a doc for a version that already shipped.

1. **Find the doc.** The highest-version file in `docs/Releases/` whose version is still ahead of (or equal to) the last actual release tag/commit — i.e. the one covering the release still in progress.
2. **Read it first**, in full — you need its existing structure to merge into.
3. **Find what's new since it was written:**
   - If the doc is already committed: `git log --follow -1 --format=%H -- <doc path>` gets the commit that last touched it; `git log <that-commit>..HEAD --oneline` lists every commit since, plus check for any current uncommitted diff on top.
   - If the doc is still uncommitted (untracked or modified, per `git status`): there's no commit boundary to anchor on, so use its filesystem mtime instead — `git log --since="<doc's last-modified timestamp>" --oneline` for commits made after it was written, plus the current uncommitted diff. If that's ambiguous (e.g. the doc was touched by an unrelated edit), ask the user which commits are new rather than guessing.
   - Gather those changes the same way as step 1 above (real diffs/files, no fabrication).
4. **Merge, don't duplicate:**
   - A new change that extends an already-documented feature → append a bullet under that feature's existing `##`/sub-list, don't re-describe the feature from scratch.
   - A new change in a genuinely new area → add a new `##` section, inserted before the `---` / Upgrade notes divider at the bottom.
   - Re-check the Upgrade notes section — new commits may add a breaking change or new required config that wasn't true before.
   - Keep the same filename/version unless the version number itself changed between rounds (e.g. manifest got bumped) — if so, `git mv` the old file to the new version name rather than leaving two files for one release.
5. **Return only the incremental bullets in chat** (what's new this round), not the full doc's bullet list again — the user already has the earlier batch filed.

## Notes

- Don't invent a `docs/Releases/` convention in a repo that clearly uses something else — **check the repo root for an existing `RELEASE_NOTES.md`/`CHANGELOG.md` first**, every time, even on repos you've already made docs for. A repo can have both: an old, abandoned root file (single stale entry, no updates in months) *and* a live `docs/Releases/` convention going forward. If a root file exists and is clearly still maintained (recent commits, multiple version entries), follow that instead — surface the conflict to the user rather than silently picking one.
- Don't commit the file yourself unless the user asks — writing it satisfies "create a release doc"; committing/pushing is a separate, confirmable action.
