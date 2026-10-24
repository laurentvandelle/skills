---
name: "LV:to-pr"
description: Creates a GitHub pull request and its Git branch from the latest written plan in the conversation. The title is a Conventional Commits subject. The body starts with the Notion spec link, then the Git branch line, then that plan in English Markdown. After the pull request exists, comments `@claude implement-plan`. Use only when the user invokes /LV:to-pr.
disable-model-invocation: true
---

Create a GitHub pull request and its Git branch from the latest written plan in this conversation. Invoking `/LV:to-pr` is the go-ahead: create the branch and the pull request immediately.

## Plan

A written plan has steps, headings, or is a Cursor plan. A loose chat is not a plan.

- No written plan: stop and say there is no plan to file.
- Several written plans: use the latest one.

## Title

One Conventional Commits subject:

- Format: `<type>(<scope>): <description>`
- Types: feat, fix, docs, refactor, perf, test, build, ci, chore, revert
- Scope is the area touched. Omit it when the area is obvious.
- English, imperative, lowercase after the colon
- No period

## Body

Start the body with the Notion spec link already in the conversation, then the Git branch line, then the plan. The link is the page URL passed to `/interview-me`, or the latest Notion page URL in the conversation when that prompt is absent. `notion.so` and `app.notion.com` URLs both count. If no Notion spec URL is present, stop and say so.

Fetch that Notion card and read its `Git branch` property. If the value is already branch text, use it. If the fetch returns a `formulaResult://` pointer, the branch text is not in the payload. Compute it from the same page with this formula:

```text
ifs(prop("Type") == "story", "feature/", prop("Type") == "bug", "fix/", prop("Type") == "tech story", "tech/", "") + prop("ID") + "-" + prop("Nom").lower().replaceAll(" ", "-")
```

- `story` → `feature/`. `bug` → `fix/`. `tech story` → `tech/`. Any other Type, including `investigation`, adds no prefix.
- `ID` is the `userDefined:ID` property on the fetch.
- Lowercase `Nom`, then replace each space with `-`. Leave every other character as it is.

If `Type`, `ID`, or `Nom` is missing, stop and say which one. Do not create the branch or the pull request.

Translate the plan into English Markdown. Keep the plan's own structure.

```markdown
Spec: <notion url>

Branch name : <Git branch>

<plan>
```

## Branch

From the open workspace's git root. The pull request is on that workspace's Git remote.

```bash
git fetch origin
git symbolic-ref --short refs/remotes/origin/HEAD
```

`origin/HEAD` is the default branch. If that ref is missing, stop and say `origin/HEAD` is unset. Do not change git config.

`{RepoName}` is the git root directory name. `{lastSegment}` is the branch name after the last `/`. The worktree path is the sibling `../{RepoName}-{lastSegment}`.

Stop before creating anything when any of these already exist. Name which ones. Do not reuse them. Do not delete them.

- local branch `refs/heads/<branch>`
- remote branch `refs/remotes/origin/<branch>`
- the sibling directory
- that path in `git worktree list`

Leave the current checkout untouched. Create the worktree from the git root:

```bash
git worktree add ../{RepoName}-{lastSegment} -b <branch> origin/HEAD
```

The empty commit exists so the branch can be pushed and the pull request can open. Do not add or change files. In the worktree:

```bash
git commit --allow-empty -m "<title>"
git push -u origin HEAD
```

Do not pass `--no-verify`. If the commit or the push fails, stop and report the error. Do not open a pull request. Leave the worktree.

## Create

The pull request base is the default branch name without the `origin/` prefix. Create the pull request even when a similar one already exists.

Write the body to a file outside the repo, then from the worktree:

```bash
gh pr create --title "<title>" --base "<default-branch>" --body-file <body-file>
```

If the remote cannot be resolved or `gh pr create` fails, stop and report the error. Leave the worktree.

## Comment

When the pull request exists, comment once. The comment body is exactly this line:

```text
@claude implement-plan
```

Take the pull request URL from the URL `gh pr create` prints.

```bash
gh pr comment <url> --body "$(cat <<'EOF'
@claude implement-plan
EOF
)"
```

If the comment is refused, leave the pull request as it is. Print the pull request URL, the error, and that command. Do not retry.

When the comment is on the pull request, reply with the pull request URL, the worktree path, and the branch name.
