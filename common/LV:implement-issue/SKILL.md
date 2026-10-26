---
name: "LV:implement-issue"
description: "Implements a GitHub issue whose body contains a plan and a `Branch name :` line. Creates a sibling git worktree and branch from origin/HEAD, codes the whole plan, runs npm run verify, commits each plan section with /LV:commit, pushes once, then opens a pull request whose body holds the /LV:qa checklist. When LV_IMPLEMENT_CI=1, works in the current checkout and writes the pull request body to $RUNNER_TEMP/pr-body.md instead of opening the pull request. Use when the user invokes /LV:implement-issue with a GitHub issue URL in Cursor or Claude Code. Also /TSM:implement-issue."
disable-model-invocation: true
---

Implement one GitHub issue. Invoking `/LV:implement-issue` with an issue URL is the go-ahead. The same file serves Cursor and Claude Code. `LV:commit` and `LV:qa` are sibling skills of this file.

CI mode is on when `LV_IMPLEMENT_CI` is `1`. Otherwise it is off. In CI mode, work in the current checkout, push, and write the pull request body to `$RUNNER_TEMP/pr-body.md`. Do not create a worktree and do not run `gh pr create`.

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

## Where to work

From the git root:

```bash
git fetch origin
git symbolic-ref --short refs/remotes/origin/HEAD
```

`origin/HEAD` is the default branch (this repo: `origin/master`). If that ref is missing, stop and say `origin/HEAD` is unset. Do not change git config.

### CI mode off

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

### CI mode on

Do not create a sibling worktree. Do not leave this checkout.

Stop if the remote branch `refs/remotes/origin/<branch>` already exists. Name it. Do not reuse it. Do not delete it.

Create the branch in this checkout:

```bash
git switch -c <branch> origin/HEAD
```

If that fails because the local branch already exists, stop and name it.

Do all later edits, `npm`, and git commands in this checkout. `$RUNNER_TEMP` must be set. If it is empty, stop and say so.

When you stop before `$RUNNER_TEMP/pr-body.md` exists, post the reason on the issue:

```bash
gh issue comment <url> --body "<reason>"
```

## Implement

Obey the repo's nearest `AGENTS.md` while editing. Implement only what the plan says.

The plan is the issue body after the branch line. Text before the first `##` is context, not a commit. A `###` belongs to the `##` above it.

Code every `##` section, in order, in the working directory from Where to work. Do not commit while coding.

Then run `npm run verify` there. Fix failures and run it again. Still failing: stop. Name the failing command. Do not commit, do not push, do not run `/LV:qa`, do not open a pull request.

## Commit

When verify passes, use the /LV:commit skill to commit each section into separate commits.

## Push

No commits on the branch: stop and say the plan changed no files. Do not push.

From the working directory, push once:

```bash
git push -u origin HEAD
```

Push failed: stop and report the error. Do not run `/LV:qa`. Do not open a pull request.

## QA checklist

Run the /LV:qa skill on the issue body already fetched. Do not execute the scenarios. Keep the checklist for the pull request body; do not post it as a separate comment. Leave the title checkbox unchecked.

## Pull request

The pull request base is the default branch name without the `origin/` prefix. Title is the issue title, unchanged. No labels, assignee, milestone, or draft.

The body has, in this order:

1. 2 to 4 English lines on what the branch changes.
2. `Closes #<n>`.
3. A `---` separator, then the full `/LV:qa` checklist, unchanged.

### CI mode off

Write the body to a file in the scratchpad (or a temporary directory outside the repo), then:

```bash
gh pr create --title "<issue title>" --base "<default-branch>" --body-file <body-file>
```

`gh pr create` failed: stop and report the error.

### CI mode on

Write the body to `$RUNNER_TEMP/pr-body.md`. Do not run `gh pr create`.

## Reply

CI mode off: reply with the pull request URL, the worktree path, and the branch name. Say the QA checklist is in the pull request body. Do not reprint it in the chat.

CI mode on: reply with `$RUNNER_TEMP/pr-body.md` and the branch name. Say the workflow opens the pull request. Do not reprint the checklist.
