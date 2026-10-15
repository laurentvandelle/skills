---
name: "TSM:qa"
description: "Writes a French QA checklist from a GitHub issue plan. Groups scenarios by each ## section with precondition, actions, expected result, and a phone marker when a device is required. Use when the user invokes /TSM:qa with a GitHub issue URL, or when /TSM:implement finishes a pull request."
disable-model-invocation: true
---

Write the QA checklist for one GitHub issue plan. Print it in the chat. Do not execute the scenarios. Do not post them to the issue or the pull request.

## Input

A GitHub issue URL, or an issue body already fetched by `/TSM:implement`.

With a URL:

```bash
gh issue view <url> --json title,number,body,url
```

No URL and no body: stop and say a GitHub issue URL is required.

The plan is the issue body after the `Branch name :` line. When that line is absent, the whole body is the plan. No steps and no headings: stop and say there is no plan to test.

## Source

Derive every scenario from the plan's expected behavior. Do not read the diff, the worktree, or the code.

Write the scenario prose in French. Keep each `##` title exactly as written in the plan. Copy UI strings from the plan verbatim, including punctuation and language.

Text before the first `##` is context. A `###` stays inside the `##` above it. One group per `##`, in plan order.

The `## Verification` group lists what the plan says `npm run verify` cannot prove. Do not add a scenario whose only step is `npm run verify`.

## Output

Start with the issue title and URL. Then the groups.

Each scenario:

```markdown
## <exact ## title>

### <short French name>
- Précondition : <state before the action>
- Actions : <what the tester does>
- Résultat attendu : <observable outcome, with UI strings verbatim>
- Téléphone
```

Add the `Téléphone` line only when the scenario needs a phone, an emulator, or a live server. Omit it otherwise.

One scenario per observable outcome. Cover the plan's locked behavior, error copy, and edge cases.

## Example

```markdown
## Locked behavior

### Oublier le serveur
- Précondition : un serveur est enregistré.
- Actions : ouvrir Serveur, toucher Oublier le serveur, confirmer `Oublier ce serveur ?`.
- Résultat attendu : l'URL, l'identifiant, le mot de passe, le nom du téléphone et le dossier sont effacés. Home affiche `Aucun serveur`.
- Téléphone
```
