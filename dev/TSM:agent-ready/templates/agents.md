# Agent instruction files

`AGENTS.md` is the instruction file for every coding harness. Humans read `README.md` and `ARCHITECTURE.md`. Agents read `AGENTS.md`.

## One body, several names

The root `AGENTS.md` is the only copy of the shared rules.

- `CLAUDE.md` is a symlink to `AGENTS.md`.
- `.github/copilot-instructions.md` is a symlink to `../AGENTS.md`.

A harness that only loads the root file follows **Routing**. Each routing line is a trigger: "Before {action}, read {path}." A list of links is not routing.

## Nested files

Every top-level directory that contains production source has its own `AGENTS.md`. The root **Routing** section names each one: "Before editing `{dir}/**`, read `{dir}/AGENTS.md`." A harness that only loads the root file follows those lines. A harness that loads nested files uses the closest `AGENTS.md`, which wins over the root file. Say that in the root file.

A child directory gets a file only when its rules differ from its parent. Feature folders that share the parent's rules do not.

The root file keeps rules that apply to every task: exact commands, absolute prohibitions, where the rest of the docs live. A nested file keeps rules that apply only there. The same rule is not written in both.

## Size and shape

The root file stays under 150 lines. Past that, the rules that matter sink into noise. A nested file stays short: prohibitions, shape, tests, and one path to a real example. Background moves to `ARCHITECTURE.md` in the same directory.

Effective instruction files give a precise role, exact commands, explicit boundaries, and examples. They do not describe the project. The project description belongs in `README.md` and `ARCHITECTURE.md`. The glossary belongs in `CONTEXT.md`.

## What to point at

- Before editing a structural directory, read that directory's `AGENTS.md`.
- Before adding a dependency or crossing a boundary, read `ARCHITECTURE.md`.
- Before naming a domain concept, read `CONTEXT.md`.
- Before writing a decision record, read `docs/adr/README.md`.
- Before treating a convention as enforced, read `docs/guardrails.md`.
- Before adding a convention after review, read `docs/maintenance.md`.

## Examples

Point at a file in the repository. Pasted code goes stale.

## When a document and the code disagree

The code is the source of truth. Report the gap. Do not invent a reconciliation in the same task.
