---
name: "LV:to-issue"
description: Creates a GitHub issue from the latest written plan in the conversation. The title is a Conventional Commits subject. The body starts with the Notion spec link, then the Git branch line, then that plan in English Markdown. Use only when the user invokes /LV:to-issue. Also /TSM:to-issue.
disable-model-invocation: true
---

Create a GitHub issue from the latest written plan in this conversation. Invoking `/LV:to-issue` is the go-ahead: create the issue immediately.

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

Fetch that Notion card and read its `Git branch` property. If the fetch returns a formula pointer instead of the branch text, stop and say so.

Translate the plan into English Markdown. Keep the plan's own structure.

```markdown
Spec: <notion url>

Branch name : <Git branch>

<plan>
```

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
