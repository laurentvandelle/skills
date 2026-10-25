---
name: "LV:agent-ready-node-deps"
description: Adds an npm run verify script to a TypeScript Node repository, chaining ESLint, tsc --noEmit, dependency-cruiser, and npm test. Writes eslint.config.mjs (no barrel or relative imports) and .dependency-cruiser.cjs from that repo's module boundaries. Use when the user invokes /LV:agent-ready-node-deps, or asks to add npm run verify, dependency-cruiser, or the agent-ready Node dependency checks. Also /LV:agent-redy-node-deps and /TSM:agent-ready-node-deps.
disable-model-invocation: true
---

# Node verify pipeline

Install the checks `LV:agent-ready` leaves out. Work in the repository the user named. If they named none, use the current workspace. Do not commit unless asked. Do not edit `AGENTS.md`, `ARCHITECTURE.md`, or `CONTEXT.md`.

Read [reference.md](reference.md) before writing a config file. Fill every path, package name, and alias from this repo.

When the four scripts, the import rule, and the documented forbidden rules already exist, leave them. Run `npm run verify` and report.

## Stop

Stop before editing when any of these is true. Name which one.

- No root `package.json`.
- No TypeScript program: no `tsconfig.json` and no `.ts` or `.tsx` application source.
- More than one of `package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`.
- `lint` exists and its command is not ESLint.
- `typecheck` exists and its command is not `tsc`.
- `boundaries` exists and its command is not `depcruise`.

A missing script is not a stop.

## Inspect

Record:

- Package manager, from the lockfile. No lockfile means npm.
- Workspace globs, or a single package.
- Production source directories. Application TypeScript. Omit `node_modules`, `dist`, `build`, `coverage`, `docs`, and deploy-only trees.
- Every `tsconfig*.json`: which program it typechecks, and every `paths` entry.
- Documented dependency direction in `ARCHITECTURE.md` or the root `AGENTS.md` boundaries section.
- Each workspace package name, and the dependencies that `package.json` already declares.
- The test filename suffix, and any fake or harness directory.
- Whether any package depends on `react`.
- Generated files the repo already ignores.

The allowed import form is the `paths` scheme already in the tsconfigs. When no scheme exists, use `@/{segment}/*` for each production source root: `@/domain/*` → `packages/domain/src/*`, `@/web/*` → `web/src/*`, or `@/*` → `src/*` when the repo has a single source root.

## Dependencies

Install missing packages as root devDependencies with the repo's package manager. Do not downgrade a version already in the manifest. Do not write a version by hand.

- `dependency-cruiser`
- `eslint`
- `eslint-config-prettier`
- `typescript`
- `typescript-eslint`
- `eslint-plugin-react-hooks` only when a package depends on `react`

`@eslint/js` ships with ESLint. Add it as a direct devDependency only when the config import fails.

## Scripts

Keep a script whose command already does the job. Add only a missing script.

New `lint`:

```json
"lint": "eslint"
```

New `typecheck` is `tsc --noEmit` when one root `tsconfig.json` typechecks the program. When each workspace has its own program, chain `tsc --noEmit -p <path>` for every production tsconfig. When the root config is only a solution (`files: []` and `references`), use `tsc --noEmit -b`.

New `boundaries` scans the production source directories, space-separated:

```json
"boundaries": "depcruise <dirs> --config .dependency-cruiser.cjs --ts-pre-compilation-deps"
```

Leave an existing root `test` script as it is. When it is missing, aggregate the workspace `test` scripts that already exist (`vitest run`, `jest`, `node --test`, or `npm test -w <name>`). Do not install a test framework. When there is no suite and no runner, stop before writing `verify` and say that `npm test` does not exist.

New `verify`:

```json
"verify": "npm run lint && npm run typecheck && npm run boundaries && npm test"
```

Do not add `format:check`. When `verify` already exists, append any missing `npm run lint`, `npm run typecheck`, `npm run boundaries`, or `npm test`. Leave steps that are already there, in their current order.

The script text uses `npm run` so `npm run verify` stays the command under pnpm and yarn too.

## ESLint

Edit the existing flat config. When none exists, create `eslint.config.mjs`. Do not add a second config. When an eslintrc file exists, fold its rules into the flat config and delete the eslintrc file. When a rule cannot be folded, stop and name it.

Follow the shape in reference.md.

- Ignores: `node_modules`, build output, and generated files found during inspect. Ignore `eslint.config.mjs`.
- `eslint.configs.recommended`, then `typescript-eslint` `recommendedTypeChecked`, with `projectService` and `tsconfigRootDir: import.meta.dirname`.
- `@typescript-eslint/no-unused-vars` allows a leading `_` on args, vars, and caught errors.
- `no-restricted-imports` on the production source directories from inspect. Tooling configs (`vite.config.ts`, `vitest.config.ts`) stay off this rule.
- Forbid each workspace package name (the barrel) and relative specifiers. Forbid `@scope/*` for each workspace scope. Forbid `#/*` and `#/**` unless `#` is the allowed alias.
- External packages stay allowed. Do not use a `*` group.
- A second `no-restricted-imports` block only for directories a documented boundary bars from persistence or the composition root. Name the modules that boundary names. Do not invent the block.
- `eslint-plugin-react-hooks` flat recommended config on the React source globs, only when React is a dependency. Place it before Prettier.
- `eslint-config-prettier` last.

Messages use the language of `AGENTS.md` or `README.md`. English when those files are absent. Wording is in reference.md.

When `recommendedTypeChecked` reports a throw type a framework already uses for control flow, allow that class. Add the allow only after lint names the type. The shape is in reference.md.

## Dependency cruiser

Edit `.dependency-cruiser.cjs`, or create it. `severity` is `error`. Each comment says why the edge is forbidden, in the repo's language.

Encode a forbidden edge when a document states it, or when every `package.json` already respects it: the inner package does not declare that dependency, and application source does not import it. A document wins when the two disagree. Do not forbid a dependency the inner package already declares unless a document forbids it.

Rules:

1. One isolation rule per inner layer. `from` is that tree. `to` is the sibling trees it must not import, plus framework packages the document names.
2. One `{layer}-prod-no-test-kit` rule per production tree that sits next to tests. `pathNot` is the repo's test suffix and its fake or harness directory. `to` is that test runner and that directory. Omit the fake path when the directory does not exist.
3. One UI rule when a document bars those files from persistence or the composition root. `pathNot` keeps the exceptions the document names.

`from.path` and `to.path` are regular expressions against paths relative to the repo root, as in reference.md.

Copy the `options` block from reference.md. Point `doNotFollow` and `exclude` at `node_modules`, build output, and the generated files found during inspect. Set `tsConfig.fileName` to the root tsconfig that maps every allowed alias. When no file does, create `tsconfig.boundaries.json` from reference.md and point at it. The `typecheck` script does not use that file.

Add `jsx` to that tsconfig only when the repo uses JSX.

Add the same cross-package `paths` to each program tsconfig that compiles a file which imports them. One folder has one alias.

## Imports

Run ESLint. For each `no-restricted-imports` hit, rewrite the specifier to the allowed alias. Keep the imported bindings. A barrel becomes the file that declares the binding. When that file is not obvious, stop and name the import. Do not guess an `index`.

Leave generated files ignored. Do not rewrite them.

## Verify

Run `npm run verify` from the root.

- Config or resolution error: fix the config or the alias map, then run it again.
- Restricted import: rewrite it, then run it again.
- `only-throw-error` for a framework control-flow throw: add the allow, then run it again.
- A forbidden dependency the source really uses, and a document forbids it: stop. Leave the rule. Name the file and the rule.
- A forbidden dependency encoded only from `package.json`, which the source already imports: remove that rule, list it under **Not encoded**, then run it again.
- Any other failure this config introduced: fix the code, then run it again.

Leave a documented rule in place. Hand back only after `npm run verify` passes, or after the stop above.

## Report

Omit empty sections.

### Scripts
### Installed
### Config
### Verify
### Not encoded

**Verify** is the command and that it passed, or the rule and file where you stopped. **Not encoded** is one line per boundary you saw and did not turn into a rule, with the reason.
