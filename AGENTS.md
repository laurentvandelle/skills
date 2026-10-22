# Agent practices for this skills repo

This repository is the personal Cursor skills library (`~/.cursor/skills`). Follow these rules when adding or changing a skill.

## Layout

```
<category>/LV:<skill-name>/
├── SKILL.md              # required
├── reference.md          # optional, loaded only when needed
└── scripts/              # optional
```

- Group skills by domain (`dev/`, `helpers/`, `notion/`). Do not put skills at the repo root.
- One skill per directory. The directory name **must** match the YAML `name`.
- Never create or edit skills in `~/.cursor/skills-cursor/` (Cursor built-ins).

## Naming

Every skill is `LV:<skill-name>`:

- `name` in frontmatter: `"LV:code-review"` (always quote — the colon is YAML-significant)
- Folder: `dev/LV:code-review/`
- Slash command: `/LV:code-review`

`<skill-name>` is kebab-case, specific, max 64 characters for the full `LV:…` id. No generic names (`helper`, `utils`, `tools`).

Keep old aliases in the **description** so `/summary` and `/TSM:summary` still find `LV:summary`. Do not keep a second unprefixed copy of the skill.

## SKILL.md

```markdown
---
name: "LV:example"
description: What it does. Use when the user invokes /LV:example, or <trigger terms>.
disable-model-invocation: true
---
```

| Field | Rule |
|-------|------|
| `name` | `"LV:<skill-name>"`, quoted |
| `description` | Third person. WHAT + WHEN. Include `/LV:<skill-name>` and any legacy triggers. Max 1024 chars. |
| `disable-model-invocation` | Default `true` (explicit invoke only). Omit only when the skill should auto-apply from ambient context. |

Body:

- Under 500 lines. Assume the agent is already smart — only write what it would not infer.
- Progressive disclosure: essentials in `SKILL.md`, details one level deep in sibling files.
- Concrete steps, templates, and examples. No time-sensitive dates. One term per concept.
- Forward slashes in paths (`scripts/helper.py`).
- If the skill wraps another, name the target with its `LV:` id (see `LV:interview-me` → `LV:interviewing`).

## Adding a skill

1. Confirm no existing `LV:` skill already covers the job.
2. Pick the category folder. Create `LV:<skill-name>/SKILL.md`.
3. Write the description with triggers before writing the body.
4. Add scripts only when generated code would be fragile or inconsistent.
5. Check: name = folder, description has WHAT+WHEN, body < 500 lines, links are one level deep.

## Maintaining a skill

- Rename as a unit: folder, `name`, slash-command mentions, and cross-skill references.
- Do not duplicate a skill under the old unprefixed folder after a rename.
- Leave vendored internals alone unless they encode **this** skill’s identity. Example: `LV:impeccable` still calls `.cursor/skills/impeccable/` because that is the **project-local install path**, not this repo’s folder.
- Prefer tightening instructions over adding prose. Delete stale steps.

## Do not

- Invent a skill without a clear trigger and a task the agent cannot already do well.
- Auto-invoke a skill that should only run on `/LV:…` (`disable-model-invocation: true`).
- Nest reference files more than one level from `SKILL.md`.
- Commit secrets, tokens, or Notion/workspace credentials inside a skill.
