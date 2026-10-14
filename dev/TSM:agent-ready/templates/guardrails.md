# Executable guardrails

Documents steer. Checks constrain. An agent follows a rule that fails the build.

## Where a convention lives

| Kind of convention | Where it lives |
| --- | --- |
| Layer boundaries, forbidden imports, strict types, mechanical naming | An executable check that already exists |
| Semantic names, slicing, product language | The nearest `AGENTS.md`, or `CONTEXT.md` for glossary terms |

Name the check in the root `AGENTS.md` command table. When the repository has a script named `verify`, the definition of done is that command:

```markdown
Run `{exact command}` before handing work back.
Never hand back a failure: fix it, then run the command again.
```

## What this convention does not do

Making a repository agent-ready records the `verify` script that already exists. It does not add a script, a linter, or a boundary tool. Those are separate changes, reviewed on their own.

When no `verify` script exists, the instruction file does not invent one. The gap stays visible. `test` or `lint` alone is not a definition of done.

## Mechanisms teams use later

When a team decides to encode a rule, typical tools are `dependency-cruiser` or `eslint-plugin-boundaries` for layer borders, `no-restricted-imports` for forbidden imports, `tsc --strict` for types, and a dedicated lint rule for mechanical naming. Install and configuration belong to that change, not to these instruction files.

After the check exists, point `AGENTS.md` at the single command that runs it. Do not copy the rule into prose as well.
