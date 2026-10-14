---
name: "TSM:agent-ready"
description: Sets up harness-agnostic coding-agent conventions in a repository. Creates or completes nested AGENTS.md files, a root ARCHITECTURE.md, a CONTEXT.md glossary, docs/ method guides, and docs/adr/. Use when the user invokes /TSM:agent-ready, or asks to make a repository agent-ready, add AGENTS.md conventions, or set up CONTEXT.md, architecture docs, and ADRs for coding agents.
disable-model-invocation: true
---

# Agent-ready repository

Install one instruction tree that any coding harness can follow. `AGENTS.md` is the only instruction body. Root aliases cover harnesses that look for another filename. Method guides live in `docs/`. The glossary lives in `CONTEXT.md`. Background lives in `ARCHITECTURE.md`.

Work in the repository the user named. If they named none, use the current workspace. Do not commit unless asked. Do not edit package manifests, lockfiles, or linter config.

Do not create, edit, or delete editor-specific rule files. Do not mention them in the report.

## Files

| Path | Role |
| --- | --- |
| `AGENTS.md` | Shared rules, exact commands, routing. Under 150 lines when this skill creates it. |
| `<dir>/AGENTS.md` | One file in every top-level production directory, plus a child that is its own boundary. Under 80 lines. |
| `ARCHITECTURE.md` | Background. Always at the root. |
| `<structural>/ARCHITECTURE.md` | Background for one directory, only when that directory has background a short instruction file cannot hold. |
| `CONTEXT.md` | Glossary. No implementation. |
| `CLAUDE.md` | Symlink to `AGENTS.md`. |
| `.github/copilot-instructions.md` | Symlink to `../AGENTS.md`. |
| `docs/agents.md` | Method guide. Copied from the template. |
| `docs/guardrails.md` | Method guide. Copied from the template. |
| `docs/maintenance.md` | Method guide. Copied from the template. |
| `docs/adr/README.md` | When and how to write an ADR. Copied from the template. |

Templates sit next to this skill: `~/.cursor/skills/dev/TSM:agent-ready/templates/`. Copy them verbatim when the destination file is missing.

## Inspect

Read the root manifest, the lockfile, the top of `README.md`, existing `AGENTS.md` files, `ARCHITECTURE.md` files, `CONTEXT.md`, and `CONTEXT-MAP.md` if present. List the top of the tree. Read an existing boundary config only to name the check that already runs.

Record:

- The package manager or task runner, from the lockfile or the README.
- Scripts that exist. A verify command is a script named `verify`. If none exists, there is no verify command. `test`, `lint`, `check`, and `ci` stay in the command table and do not become a definition of done.
- Structural directories (below).
- Domain terms whose meaning the code or an existing doc states in words.
- Statements in existing docs that contradict the code. Those are gaps.

## Structural directories

Create an `AGENTS.md` in every top-level directory that contains production source. A short file still gets written. Do not skip a directory because its conventions look thin or obvious.

Production source is application code. These top-level directories stay without an `AGENTS.md`:

- dependencies, build output, and generated trees (`node_modules`, `dist`, `build`, `coverage`, `vendor`)
- documentation only (`docs`)
- editor, git, and CI metadata (`.git`, `.github`, `.cursor`)
- a directory whose files are tests and nothing else

Also create an `AGENTS.md` in a workspace member below the root, and in one child of a structural directory when that child is its own boundary: forbidden imports, a test layout, or composition rules that differ from the parent. A child that only groups features sharing the parent's rules does not get a file. When a child is unclear, leave it out and list it under **Left unchanged**. A top-level production directory is never left out.

Creating the file and pointing at it are one step. For every nested `AGENTS.md`, the root **Routing** section contains:

```markdown
- Before editing `{dir}/**`, read `{dir}/AGENTS.md`.
```

A root file that names a directory as a boundary, without that bullet and without the nested file, is incomplete. Add both before finishing.

## Merge

For every file this skill owns:

- Missing file: create it.
- Existing file: add missing sections and missing bullets. Do not delete or rephrase a rule that is already there.
- A statement that contradicts the code stays. Add a **Gaps** entry. Do not "fix" it inside this pass.
- An existing root `AGENTS.md` already over 150 lines stays over the limit. Report the line count. Do not cut the owner's text to meet the cap.
- A new root `AGENTS.md` must be under 150 lines. A new nested `AGENTS.md` must be under 80 lines. Move background into `ARCHITECTURE.md` to get there.

## Write the repository files

### ARCHITECTURE.md

Write from the code. Omit empty sections. Use this shape:

```markdown
# Architecture

Operational rules for agents live in `AGENTS.md`. This file is background.

## Intent

## Modules

## Boundaries

## Runtime

## Out of scope
```

Name modules, allowed dependency direction, and runtime composition only when the repository actually shows them. In a structural directory, add `ARCHITECTURE.md` only when the background is real (a dependency diagram, a composition root, invariants that need prose). The local `AGENTS.md` then points at that sibling file.

### Nested AGENTS.md

```markdown
# AGENTS.md — {directory}

Background: `ARCHITECTURE.md` in this directory.

## Prohibitions

## Shape

## Tests

## Example
```

Drop **Background** when the directory has no `ARCHITECTURE.md`. Drop empty sections. Prohibitions are local. **Example** is a path to a real file in that directory, or the section is omitted.

### Root AGENTS.md

```markdown
# AGENTS.md

Instructions for every coding agent. Humans read `README.md` and `ARCHITECTURE.md`.

## Role

## Commands

## Routing

## Non-negotiables

## Boundaries

## Definition of done

## Maintenance
```

- **Role** — a few lines on what this repository is for. No tour of the tech stack.
- **Commands** — a table of commands that exist, each with the exact invocation (`npm run verify`, `pnpm test`, `make check`). Point local setup at `README.md`.
- **Routing** — one bullet per trigger, in the form "Before {action}, read {path}." Directory bullets come first, one per nested `AGENTS.md`, using the sentence in **Structural directories**. Also include, when the file exists:
  - Before adding a dependency or crossing a module boundary, read `ARCHITECTURE.md`.
  - Before naming a domain concept, read `CONTEXT.md`.
  - Before writing an architecture decision, read `docs/adr/README.md`.
  - Before changing these instruction files, read `docs/agents.md`.
  - Before treating a convention as enforced, read `docs/guardrails.md`.
  - Before adding a convention after review, read `docs/maintenance.md`.
- **Non-negotiables** — ensure these three bullets exist, without duplicating them if the wording is already there:
  1. The `AGENTS.md` closest to the file being changed wins over the root file.
  2. The code is the source of truth. When a document and the code disagree, report the gap. Do not invent a reconciliation in the same task.
  3. A rule lives in one file. Local rules live in the nearest `AGENTS.md`. The root file keeps shared rules only.
- **Boundaries** — only when an import direction is visible. Name the existing check command when one exists.
- **Definition of done** — only when a `verify` script exists. Text:

  ```markdown
  Run `{exact command}` before handing work back.
  Never hand back a failure: fix it, then run the command again.
  ```

- **Maintenance** — two lines pointing at `docs/maintenance.md`.

Shared rules stay at the root. Local rules stay nested. Do not paste a method guide into `AGENTS.md`.

### CONTEXT.md

Follow this format. Prose is English. The headword is the word the project already uses, including when that word is not English.

```markdown
# {Context name}

{One or two sentences: what this context is, and why it exists.}

## Language

**{Term}**:
{One or two sentences. What it is. Not what the code does with it.}
_Avoid_: {other words used for the same concept}
```

Rules:

- A glossary and nothing else. No frameworks, databases, folders, or API shapes.
- Include a term only when the code or an existing document states its meaning. A type name alone is not a meaning.
- Skip general programming words.
- `_Avoid_` lists aliases the repository actually uses. Omit the line when there is no alias.
- Group terms under subheadings when clusters are obvious. Otherwise keep a flat list.
- Omit `## Language` when no term qualifies.
- When `CONTEXT-MAP.md` exists, follow it and edit the context file it names. Do not invent a second context, and do not create a map.
- Ambiguous candidates go in the report only.

### Method guides and ADR index

Copy from the templates when missing:

| Template | Destination |
| --- | --- |
| `templates/agents.md` | `docs/agents.md` |
| `templates/guardrails.md` | `docs/guardrails.md` |
| `templates/maintenance.md` | `docs/maintenance.md` |
| `templates/adr-README.md` | `docs/adr/README.md` |

When the destination exists, keep its extra text and add any missing headings from the template. Do not replace the file. Do not create an ADR. An ADR file appears only when a later change meets the three tests in that README.

### Aliases

From the repository root, after `AGENTS.md` exists:

- `CLAUDE.md` missing: `ln -s AGENTS.md CLAUDE.md`
- `.github/copilot-instructions.md` missing: create `.github` when needed, then `ln -s ../AGENTS.md .github/copilot-instructions.md`
- Already the correct relative symlink: leave it
- A real file, or a symlink to somewhere else: leave it, and say so in the report

Do not replace either file with a copy of `AGENTS.md`.

## Check before the report

Finish only when all three are true:

- Every top-level production directory has an `AGENTS.md`.
- Root **Routing** has the directory bullet from **Structural directories** for each nested file.
- **Non-negotiables** says the closest `AGENTS.md` wins.

Add whatever is missing, then report.

## Report

End with a short report. Omit empty sections. Use these headings only:

### Created
### Completed
### Left unchanged
### Gaps
### Glossary candidates
### Verify command
### Aliases

**Completed** names the sections added. **Left unchanged** is for child directories you considered and did not scaffold. **Glossary candidates** is one line per excluded term and why it is ambiguous. **Verify command** is the exact command, or "No `verify` script found. No manifest was changed." **Aliases** states each symlink created, already correct, or left in place because a real file was there.
