# skills

Personal skills for Cursor and Claude Code.

- `common/` — both agents
- `cursor/` — Cursor only
- `claude/` — Claude Code only

Each skill is `LV:<skill-name>/` directly under its harness.

## Install

Symlink each `LV:…` skill folder into the agent skills directory. The agent reads these files, so a pull or an edit is the update. Leave `~/.cursor/skills-cursor/` alone — that folder is Cursor's built-ins.

```bash
REPO="$HOME/PERSO/skills"   # path to this clone
```

Edits to a linked skill are already live. Run the loop again after adding or renaming a skill. Delete the matching link when you remove one.

### Cursor

`~/.cursor/skills/` applies in every project. Link every skill folder from `common/` and `cursor/`.

```bash
mkdir -p ~/.cursor/skills
find "$REPO/common" "$REPO/cursor" \
  -mindepth 1 -maxdepth 1 -type d -name 'LV:*' -print \
| while IFS= read -r dir; do
  ln -sfn "$dir" "$HOME/.cursor/skills/$(basename "$dir")"
done
```

If that directory still has harness symlinks from the previous install, remove them so each skill is listed once:

```bash
rm -f ~/.cursor/skills/common ~/.cursor/skills/cursor
```

The same links under a project's `.cursor/skills/` apply only in that project. A `.cursor/skills/` nested inside a subdirectory is scoped to files under that subdirectory.

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

Cursor also loads `~/.claude/skills/`. With both installs, Cursor sees each `common/` skill twice, and it sees both the `cursor/` and `claude/` copies of `LV:to-pr`, `LV:interview-me`, and `LV:interviewing`.
