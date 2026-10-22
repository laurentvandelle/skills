---
name: "LV:commit"
description: "Splits the current changes into a few Conventional Commits, one per broad area of the coding session, or a single commit when the work is one area. Use when the user invokes /LV:commit or asks to commit changes. Also /TSM:commit."
---

Commit the current changes. Group them into the broad areas of the coding session. One commit when the diff is one area. A handful at most. Do not split by file, hunk, or small fix.

If the caller already named a subset (for example one plan section), commit only that subset, still as one commit unless it mixes unrelated areas.

## Grouping

Read `git status`, `git diff` (staged and unstaged), and recent `git log` subjects in parallel before the first commit.

Name the coarse areas from what the session changed. Merge anything that belongs to the same change. Leave secrets and unrelated untracked files out.

Stage only the files for one area, commit, then the next. Commit an area that others depend on first. When one file spans two areas, keep it with the area it mainly serves.

## Message

Conventional Commits, matching recent log style when it already follows this:

- Format: `<type>(<scope>): <description>`
- Types: feat, fix, docs, refactor, perf, test, build, ci, chore, revert
- Scope = area touched (web, auth, db, domain, infra…); omit if obvious
- One short English line, imperative, lowercase after the colon
- No period, no body

Pass the message with a HEREDOC. Do not push. Do not amend.
