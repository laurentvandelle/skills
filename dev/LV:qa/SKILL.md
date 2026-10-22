---
name: "TSM:qa"
description: "Writes an English QA checklist from a GitHub issue plan. The title is an unchecked box to tick when QA is done. Groups scenarios by each ## section with a precondition, one checkbox per action step, an expected result, and a 📱 marker when a device is required. Use when the user invokes /TSM:qa with a GitHub issue URL, or when /TSM:implement-issue needs the checklist for its pull request body."
---

Write the QA checklist for one GitHub issue plan. Do not execute the scenarios.

Invoked directly: print the checklist in the chat. Do not post it to the issue or the pull request.

Invoked by `/TSM:implement-issue`: return the checklist so that skill puts it in the pull request body.

## Input

A GitHub issue URL, or an issue body already fetched by `/TSM:implement-issue`.

With a URL:

```bash
gh issue view <url> --json title,number,body,url
```

No URL and no body: stop and say a GitHub issue URL is required.

The plan is the issue body after the `Branch name :` line. When that line is absent, the whole body is the plan. No steps and no headings: stop and say there is no plan to test.

## Source

Derive every scenario from the plan's expected behavior. Do not read the diff, the worktree, or the code.

Write the scenario prose in English. Keep each `##` title exactly as written in the plan. Copy UI strings from the plan verbatim, including punctuation and language.

Text before the first `##` is context. A `###` stays inside the `##` above it. One group per `##`, in plan order.

The `## Verification` group lists what the plan says `npm run verify` cannot prove. Do not add a scenario whose only step is `npm run verify`.

## Output

Start with this header. The title is a GitHub task-list item. Leave it unchecked. The tester ticks it when the QA is done.

```markdown
- [ ] QA — <issue title>

<issue URL>

📱 = needs a phone, an emulator, or a live server.
```

Then the groups.

Each scenario:

```markdown
## <exact ## title>

### <short English name> 📱
- Precondition: <state before the action>
- Actions:
  - [ ] <first step the tester does>
  - [ ] <next step>
- Expected result: <observable outcome, with UI strings verbatim>
```

Actions are always a nested checkbox list, one `- [ ]` per step the tester performs, in order, so each step can be ticked on GitHub. A scenario with a single step still uses one checkbox. Each step is one imperative English sentence.

Add ` 📱` at the end of the scenario title only when the scenario needs a phone, an emulator, or a live server. Omit it otherwise.

One scenario per observable outcome. Split a scenario when its steps lead to different results (for example one scenario per HTTP status). Cover the plan's locked behavior, error copy, and edge cases.

## Example

```markdown
- [ ] QA — Forget a registered server

## Locked behavior

### Forget the server 📱
- Precondition: a server is registered.
- Actions:
  - [ ] Open Server and tap **Oublier le serveur**.
  - [ ] Confirm `Oublier ce serveur ?`.
  - [ ] Return to Home.
- Expected result: the URL, identifier, password, phone name, and folder are cleared. Home shows `Aucun serveur`.
```
