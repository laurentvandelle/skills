---
name: "TSM:to-issue"
description: Creates a GitHub issue from the latest written plan in the conversation. The title is a Conventional Commits subject and the body is that plan in English Markdown. Use only when the user invokes /TSM:to-issue.
disable-model-invocation: true
---

Create a GitHub issue from the latest written plan in this conversation. Invoking `/TSM:to-issue` is the go-ahead: create the issue immediately.

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

Translate the plan into English Markdown. Keep the plan's own structure. The body is only that plan.

## Create

From the open workspace, create the issue on that workspace's Git remote. Set no labels, assignee, or milestone. Create the issue even when a similar one already exists.

```bash
gh issue create --title "<title>" --body "$(cat <<'EOF'
<body>
EOF
)"
```

Reply with the issue URL.

If the remote cannot be resolved or `gh issue create` fails, stop and report the error.
