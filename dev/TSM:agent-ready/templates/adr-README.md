# Architecture decision records

ADRs in this directory use sequential names: `0001-slug.md`, `0002-slug.md`.

Write an ADR only when all three are true:

1. **Hard to reverse** — changing your mind later has a real cost.
2. **Surprising without context** — a later reader will wonder why it was done this way.
3. **A real trade-off** — there were genuine alternatives, and one was chosen for specific reasons.

If any test fails, skip the ADR.

## Template

```markdown
# {Short title of the decision}

{1–3 sentences: the context, the decision, and why.}
```

An ADR can be a single paragraph. The value is recording that a decision was made, and why.

## Optional sections

Add these only when they earn a place. Most ADRs need none of them.

- **Status** frontmatter (`proposed | accepted | deprecated | superseded by ADR-NNNN`) when a decision is revisited
- **Considered options** when a rejected alternative is worth remembering
- **Consequences** when a downstream effect is not obvious

## Numbering

Scan this directory for the highest number and add one. `README.md` is not an ADR.

## What qualifies

- Architectural shape.
- How contexts integrate.
- A technology choice that carries lock-in. Not every library.
- A boundary or a scope decision, including an explicit no.
- A deliberate deviation from the obvious path.
- A constraint that is invisible in the code.
- A rejected alternative whose rejection was not obvious.

If a decision is easy to reverse, skip it. If nobody would wonder why, skip it. If there was no real alternative, there is nothing to record beyond doing the obvious thing.
