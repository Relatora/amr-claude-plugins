---
name: detailed-commit
description: Draft and create a detailed git commit message (subject line + bullet list of concrete changes + optional "Issue #NN" footer). Takes an issue number; if none is given or found in context, asks the user for one — including an explicit "no issue" option — before drafting. Use when the user says "detailed commit", "write a detailed commit message", "commit this for issue #NN", or invokes /detailed-commit.
---

# detailed-commit

Create a commit whose message actually documents what changed — not a one-line Conventional Commit summary.

## 1. Get the issue number

Check, in order: an issue number passed as an argument to this skill, one mentioned earlier in the conversation, or one visible in the current branch name. If none of those give you a number, ask the user directly before drafting anything — offer "no issue for this one" as an explicit option alongside entering a number, don't just leave the prompt open-ended. Do not guess or invent a number. If the user picks no issue, drop the `Issue #<N>` footer entirely — the message ends after the subject/bullets, no blank trailing line for it.

## 2. Gather the real changes — never fabricate

Run in parallel: `git status`, `git diff` (unstaged) and `git diff --staged` (staged), `git log --oneline -15`.

If something is already staged, describe the staged diff — don't mix in unrelated unstaged noise, and skip straight to step 3.

If nothing is staged but the working tree has uncommitted changes (modified, deleted, or untracked files), ask the user via AskUserQuestion whether to stage everything and commit it all as one commit, or hold off so they can stage what they want first. Offer exactly these two options: "Stage everything and commit it all" and "I'll stage what I want myself". If the user doesn't answer — declines the question, or the session is non-interactive — default to the first option: stage everything and commit it all. "Stage everything" means adding every file shown in `git status` by name (never `-A`/`.` — see step 4), not skipping files because they look like a separate concern; splitting into multiple commits only happens when the user asks for it or picks the second option and stages a subset themselves.

Read any changed file whose diff hunk alone doesn't explain the "why" or full scope — describe what actually changed, not just what the patch lines say out of context.

Every bullet in the message must map to a real hunk you looked at. If a change is a dependency bump, name the package and old→new version.

## 3. Match the repo's own message shape

Look at *this* repo's own recent `git log` first for an existing commit-message convention (bullet-list bodies, an issue-footer style, something else entirely) and follow whatever it already does — don't impose a shape the repo doesn't use.

If the repo's history doesn't show a clear existing pattern (a fresh repo, or one with only terse one-line messages so far), default to this shape:

```
<type>: <subject — imperative, specific, can run long, no length cap>
- <bullet: one concrete change, specific enough to mean something on its own>
- <bullet: another concrete change>
- <as many bullets as there are distinct real changes — don't pad to a count>

Issue #<N>
```

Rules for the default shape:
- `<type>` is Conventional-Commits-style (`feat:`, `fix:`, `refactor:`, `chore:`, `docs:`) — pick whichever matches what the diff actually is, or match the repo's own prefix style if it already uses one.
- Bullets are optional for a genuinely small, single-purpose change — a subject line alone plus the `Issue #<N>` footer is fine when there's nothing more to say. Don't invent filler bullets to look thorough.
- The `Issue #<N>` line is always last, preceded by exactly one blank line, exact casing `Issue #<N>` (capital I, no colon).
- No other footer content (no "Co-authored-by", no generated-by trailers) unless the user asks for one.

## 4. Stage, commit, verify

Staging scope was already decided in step 2 — either the user staged things themselves, or this skill staged every changed/untracked file by name per the default there. Stage by explicit filename only (never `-A`/`.`), even when the scope is "everything" — list every file `git status` showed. Pass the commit message via a heredoc so formatting survives. Never use `--no-verify`/`--amend` unless asked. After committing, run `git status` to confirm a clean result. If a pre-commit hook fails, fix the issue, re-stage, and make a new commit — don't amend.

## 5. Push

After a clean commit, push it: `git push`. If the branch has no upstream yet, use `git push -u origin <branch>` instead. Never force-push (`--force`/`-f`) unless the user explicitly asks for it.
