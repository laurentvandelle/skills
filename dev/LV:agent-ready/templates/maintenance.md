# Keeping instructions true

Instruction files are code. They are versioned, reviewed in pull requests, and tested by use. Without upkeep they go stale in weeks, and the team stops trusting them.

## When a review finds a violation

1. If a tool can reject the violation, add that check in its own change. Then point the root `AGENTS.md` at the command that already runs it.
2. Otherwise add one rule to the `AGENTS.md` closest to the code that broke the convention.
3. Leave the root file for rules that apply everywhere. Do not paste a local rule in both places.

## Drift

When a document and the code disagree, report the gap. Update the document or the code in a deliberate change. Do not paper over the disagreement inside an unrelated task.

## Size

A root `AGENTS.md` past 150 lines is burying the rules that matter. Move local rules into the nearest nested `AGENTS.md`. Move background into `ARCHITECTURE.md`.
