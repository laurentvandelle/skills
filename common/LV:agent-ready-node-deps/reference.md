# Config shapes

Fill every `{placeholder}` from the repo. Messages follow the language of `AGENTS.md` or `README.md`.

## ESLint

`eslint.config.mjs`:

```javascript
// @ts-check

import eslint from '@eslint/js';
import prettier from 'eslint-config-prettier';
import tseslint from 'typescript-eslint';
// import reactHooks from 'eslint-plugin-react-hooks';

const noBarrelOrRelative = {
  paths: [
    { name: '{package-name}', message: '{Import the file via @/segment/…}' },
  ],
  patterns: [
    {
      group: ['{scope}/*', './*', '../*', '../**', './**', '#/*', '#/**'],
      message: '{No relative import and no package barrel: use the @/ aliases.}',
    },
  ],
};

export default tseslint.config(
  {
    ignores: [
      '**/node_modules/**',
      '**/dist/**',
      '**/.output/**',
      'eslint.config.mjs',
    ],
  },
  eslint.configs.recommended,
  tseslint.configs.recommendedTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      '@typescript-eslint/no-unused-vars': [
        'error',
        { argsIgnorePattern: '^_', varsIgnorePattern: '^_', caughtErrorsIgnorePattern: '^_' },
      ],
    },
  },
  {
    files: ['{src}/**/*.{ts,tsx}'],
    rules: {
      'no-restricted-imports': ['error', noBarrelOrRelative],
    },
  },
  prettier,
);
```

Drop `#/*` and `#/**` from `group` when `#` is the allowed alias. One `paths` entry per workspace package that has an alias. One `group` prefix per workspace scope (`@scope/*`). An unscoped package name goes in `paths` only.

`files` is application source. Do not add `vite.config.ts` or `vitest.config.ts`.

### UI source

Add this block only for a documented boundary. It replaces `noBarrelOrRelative` for those files, so repeat the barrel paths and extend the pattern.

```javascript
{
  files: ['{ui-dirs}/**/*.{ts,tsx}'],
  ignores: ['{documented-exceptions}'],
  rules: {
    'no-restricted-imports': [
      'error',
      {
        paths: [
          ...noBarrelOrRelative.paths,
          { name: '{orm-or-client}', message: '{Persistence stays in the db package.}' },
        ],
        patterns: [
          {
            group: [
              '{scope}/*',
              '{forbidden-alias}',
              '{forbidden-alias}/**',
              './*',
              '../*',
              '../**',
              './**',
              '#/*',
              '#/**',
            ],
            message: '{UI uses its alias. No persistence and no composition root.}',
          },
        ],
      },
    ],
  },
},
```

French wording when the repo is French:

- Barrel path: `Importer le fichier via @/{segment}/…`
- Pattern: `Pas d’import relatif ni @scope/* : utiliser {aliases}.`
- UI pattern: `Pas de persistance ni de composition root.`

### React

Before `prettier`, only when React is a dependency:

```javascript
{
  files: ['{react-src}/**/*.{ts,tsx}'],
  ...reactHooks.configs.flat.recommended,
},
```

### Framework throws

Add inside the shared `rules` object only after lint names the type:

```javascript
'@typescript-eslint/only-throw-error': [
  'error',
  {
    allow: [
      { from: 'package', name: ['{ClassName}'], package: '{package}' },
    ],
  },
],
```

## dependency-cruiser

`.dependency-cruiser.cjs`:

```javascript
/** @type {import('dependency-cruiser').IConfiguration} */
module.exports = {
  forbidden: [
    {
      name: '{layer}-isolated',
      comment: '{Inner layer imports no framework, ORM, or other workspace package.}',
      severity: 'error',
      from: { path: '^{inner-dir}' },
      to: {
        path: '(^{sibling-dir}(/|$))|(node_modules/({framework|packages}))',
      },
    },
    {
      name: '{layer}-prod-no-test-kit',
      comment: '{Production source imports neither the test runner nor the fakes.}',
      severity: 'error',
      from: {
        path: '^{inner-dir}/src',
        pathNot: '({test-suffix})|(^{fake-dir})',
      },
      to: {
        path: '(node_modules/{test-runner})|(^{fake-dir})',
      },
    },
  ],
  options: {
    doNotFollow: {
      path: '(^node_modules)|({generated})',
    },
    exclude: {
      path: '(^node_modules)|({generated})|(^dist)',
    },
    moduleSystems: ['es6', 'cjs'],
    tsPreCompilationDeps: true,
    combinedDependencies: true,
    tsConfig: {
      fileName: '{tsconfig.boundaries.json|tsconfig.json}',
    },
    enhancedResolveOptions: {
      exportsFields: ['exports'],
      conditionNames: ['import', 'require', 'node', 'default', 'types'],
      extensions: ['.ts', '.tsx', '.js', '.jsx', '.mjs', '.cjs'],
    },
  },
};
```

`path` values are regular expressions on paths relative to the repo root. `{test-suffix}` matches the repo's test files, for example `\\.unit\\.test\\.ts$` or `\\.test\\.tsx$`. Drop the framework alternative from `to.path` when the document does not name frameworks. Drop the fake alternative when that directory does not exist.

UI rule, only when a document bars those files from persistence or the composition root:

```javascript
{
  name: '{ui}-no-persistence',
  comment: '{UI does not import persistence or the composition root.}',
  severity: 'error',
  from: {
    path: '^{ui-dir}/',
    pathNot: '^{documented-exception}',
  },
  to: {
    path: '(^{persistence-dir}(/|$))|(^{composition-root})|(node_modules/({orm|client}))',
  },
},
```

## Alias map

Create `tsconfig.boundaries.json` only when no root tsconfig already maps every allowed alias. dependency-cruiser reads it. `typecheck` does not.

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noEmit": true,
    "skipLibCheck": true,
    "paths": {
      "@/{segment}/*": ["./{source}/*"]
    }
  },
  "include": ["{source}"]
}
```

Add `"jsx": "react-jsx"` only when the repo uses JSX. `include` lists the production source roots.
