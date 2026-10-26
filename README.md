# skills

Personal skills for Cursor and Claude Code.

- `common/` — both agents
- `cursor/` — Cursor only
- `claude/` — Claude Code only

Each skill is `LV:<skill-name>/` directly under its harness.

## Install

Symlink this repo into the agent skills directory. The agent reads these files, so a pull or an edit is the update. Leave `~/.cursor/skills-cursor/` alone — that folder is Cursor's built-ins.

```bash
REPO="$HOME/PERSO/skills"   # path to this clone
```

### Cursor

`~/.cursor/skills/` applies in every project. Cursor walks it recursively and follows a directory symlink, so a new `LV:…` folder under the linked harness shows up on the next session without another link.

```bash
mkdir -p ~/.cursor/skills
ln -sfn "$REPO/common" ~/.cursor/skills/common
ln -sfn "$REPO/cursor" ~/.cursor/skills/cursor
```

Link `common/` and `cursor/`. The `claude/` harness is the Claude Code copy of the same skill names.

If that directory already has one symlink per skill, remove those links first so each skill is listed once:

```bash
find ~/.cursor/skills -maxdepth 1 -type l ! -name common ! -name cursor -exec rm {} +
```

The same two links under a project's `.cursor/skills/` apply only in that project. A `.cursor/skills/` nested inside a subdirectory is scoped to files under that subdirectory.

### Claude Code

Claude Code loads each direct child of `~/.claude/skills/<skill-name>/`. Link every skill folder from `common/` and `claude/`. A symlink to the harness directory is a single child with no `SKILL.md` of its own, so the skills inside it stay unloaded.

```bash
mkdir -p ~/.claude/skills
find "$REPO/common" "$REPO/claude" \
  -mindepth 1 -maxdepth 1 -type d -name 'LV:*' -print \
| while IFS= read -r dir; do
  ln -sfn "$dir" "$HOME/.claude/skills/$(basename "$dir")"
done
```

Edits to a linked skill are already live. Run the loop again after adding or renaming a skill. Delete the matching link in `~/.claude/skills/` when you remove one.

Cursor also loads `~/.claude/skills/`. With both installs, Cursor sees `common/` from two places, and it sees both the `cursor/` and `claude/` copies of `LV:to-pr`, `LV:interview-me`, and `LV:interviewing`.
