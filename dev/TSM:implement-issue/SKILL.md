---
name: "TSM:implement-issue"
description: "Implements a GitHub issue whose body contains a plan and a `Branch name :` line. Creates a sibling git worktree and branch from origin/HEAD, codes the whole plan, runs npm run verify, commits each plan section with /TSM:commit, pushes once, then opens a pull request whose body holds the /TSM:qa checklist. Use when the user invokes /TSM:implement-issue with a GitHub issue URL in Cursor or Claude Code."
disable-model-invocation: true
---

Implement one GitHub issue in a new worktree. Invoking `/TSM:implement-issue` with an issue URL is the go-ahead. The same file serves Cursor and Claude Code. `TSM:commit` and `TSM:qa` are sibling skills of this file.

## Input

The argument is one GitHub issue URL, such as `https://github.com/Herovatech/HomeDav/issues/1`.

No URL: stop and say a GitHub issue URL is required.

Fetch the issue:

```bash
gh issue view <url> --json title,number,body,url
```

Read the first line that matches `Branch name : <name>`. The branch name is the text after `Branch name : `, unchanged. Spaces before the colon are part of the label.

No such line, or an empty name: stop and say the issue has no branch name.

Resolve `owner/repo` from `git remote get-url origin`. The issue URL must name that same owner and repo. Otherwise stop.

Implement the issue even when it is closed.

## Worktree

From the git root:

```bash
git fetch origin
git symbolic-ref --short refs/remotes/origin/HEAD
```

`origin/HEAD` is the default branch (this repo: `origin/master`). If that ref is missing, stop and say `origin/HEAD` is unset. Do not change git config.

`{RepoName}` is the git root directory name. `{lastSegment}` is the branch name after the last `/`. The worktree path is the sibling `../{RepoName}-{lastSegment}`.

Example: branch `feature/5-upload-new-photos-to-dated-folders` in repo directory `HomeDav` → `../HomeDav-5-upload-new-photos-to-dated-folders`.

Stop before creating anything when any of these already exist. Name which ones. Do not reuse them. Do not delete them.

- local branch `refs/heads/<branch>`
- remote branch `refs/remotes/origin/<branch>`
- the sibling directory
- that path in `git worktree list`

Create the worktree from the git root:

```bash
git worktree add ../{RepoName}-{lastSegment} -b <branch> origin/HEAD
```

Leave the current checkout untouched. Do all later edits, `npm`, and git commands in the worktree.

## Implement

Obey the repo's nearest `AGENTS.md` while editing. Implement only what the plan says.

The plan is the issue body after the branch line. Text before the first `##` is context, not a commit. A `###` belongs to the `##` above it.

Code every `##` section, in order, in the worktree. Do not commit while coding.

Then run `npm run verify` in the worktree. Fix failures and run it again. Still failing: stop. Name the failing command. Do not commit, do not push, do not run `/TSM:qa`, do not open a pull request.

## Commit

When verify passes, use the /TSM:commit skill to commit each section into separate commits.

## Push

No commits on the branch: stop and say the plan changed no files. Do not push.

From the worktree, push once:

```bash
git push -u origin HEAD
```

Push failed: stop and report the error. Do not run `/TSM:qa`. Do not open a pull request.

## QA checklist

Run the /TSM:qa skill on the issue body already fetched. Do not execute the scenarios. Keep the checklist for the pull request body; do not post it as a separate comment. Leave the title checkbox unchecked.

## Pull request

The pull request base is the default branch name without the `origin/` prefix. Title is the issue title, unchanged. No labels, assignee, milestone, or draft.

The body has, in this order:

1. 2 to 4 English lines on what the branch changes.
2. `Closes #<n>`.
3. A `---` separator, then the full `/TSM:qa` checklist, unchanged.

Write the body to a file in the scratchpad (or a temporary directory outside the repo), then:

```bash
gh pr create --title "<issue title>" --base "<default-branch>" --body-file <body-file>
```

`gh pr create` failed: stop and report the error.

## Reply

Reply with the pull request URL, the worktree path, and the branch name. Say the QA checklist is in the pull request body. Do not reprint it in the chat.
