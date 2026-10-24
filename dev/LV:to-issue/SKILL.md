---
name: "LV:to-issue"
description: Creates a GitHub issue from the latest written plan in the conversation. The title is a Conventional Commits subject. The body starts with the Notion spec link, then the Git branch line, then that plan in English Markdown. Optional model and effort arguments add exactly one claude:<model>-<effort> label after the issue exists. Use only when the user invokes /LV:to-issue. Also /TSM:to-issue.
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

Fetch that Notion card and read its `Git branch` property. If the value is already branch text, use it. If the fetch returns a `formulaResult://` pointer, the branch text is not in the payload. Compute it from the same page with this formula:

```text
ifs(prop("Type") == "story", "feature/", prop("Type") == "bug", "fix/", prop("Type") == "tech story", "tech/", "") + prop("ID") + "-" + prop("Nom").lower().replaceAll(" ", "-")
```

- `story` → `feature/`. `bug` → `fix/`. `tech story` → `tech/`. Any other Type, including `investigation`, adds no prefix.
- `ID` is the `userDefined:ID` property on the fetch.
- Lowercase `Nom`, then replace each space with `-`. Leave every other character as it is.

If `Type`, `ID`, or `Nom` is missing, stop and say which one. Do not create the issue.

Translate the plan into English Markdown. Keep the plan's own structure.

```markdown
Spec: <notion url>

Branch name : <Git branch>

<plan>
```

## Arguments

Optional words after `/LV:to-issue` or `/TSM:to-issue`. Normalize them to lowercase. Order is model, then effort. Zero, one, or two words.

- No words: model `opus`, effort `medium`.
- One word: it must be a model. Effort is `medium`.
- Two words: a model, then an effort.

Models: `opus`, `sonnet`, `haiku`.

Efforts: `low`, `medium`, `high`, `xhigh`, `max`.

`fable` and `best` are invalid. An effort with no model is invalid (`/LV:to-issue high`). Reversed order is invalid (`/LV:to-issue high sonnet`). More than two words is invalid.

Anything else stops before `gh issue create`. Print both lists. Create no issue and no label.

The label name is `claude:<model>-<effort>`, for example `claude:opus-medium`.

## Usage

- `/LV:to-issue` → `claude:opus-medium`
- `/LV:to-issue sonnet` → `claude:sonnet-medium`
- `/LV:to-issue sonnet high` → `claude:sonnet-high`
- `/LV:to-issue opus max` → `claude:opus-max`

Labels are the cross product of the three models and the five efforts (15 names). The name carries the alias only, never a model version.

## Create

From the open workspace, create the issue on that workspace's Git remote. Set no assignee or milestone. Create the issue even when a similar one already exists. Do not pass `--label` on create. Add exactly one `claude:*` label after the issue exists.

Validate the arguments first. On failure, stop with the valid models and efforts. No issue, no label.

```bash
gh issue create --title "<title>" --body "$(cat <<'EOF'
<body>
EOF
)"
```

If the remote cannot be resolved or `gh issue create` fails, stop and report the error.

Take the issue number from the URL `gh issue create` prints. Look up the exact label name. `--search` filters server-side, so a long label list cannot hide it.

```bash
gh label list --search "<name>" --limit 100 --json name --jq '.[] | select(.name=="<name>") | .name'
```

If that command prints nothing, create the label. Do not pass `--force`, and do not change an existing label.

```bash
gh label create "<name>" --color 6E40C9 --description "Claude <model>, effort <effort>"
```

If creation is refused, leave the issue unlabeled. Print the issue URL, the error, and these commands. Do not retry.

```bash
gh label create "<name>" --color 6E40C9 --description "Claude <model>, effort <effort>"
gh issue edit <number> --add-label "<name>"
```

If creation fails because the label already exists, add it. When the label exists, or after creating it, add it:

```bash
gh issue edit <number> --add-label "<name>"
```

If adding the label is refused, leave the issue as it is. Print the issue URL, the error, and `gh issue edit <number> --add-label "<name>"`. Do not retry.

When the label is on the issue, reply with the issue URL and the label name.
