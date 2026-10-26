---
name: "LV:summary"
description: Produces a structured handoff summary of the current conversation — decisions, changes made, and pending work — formatted as copy-paste context for another agent. Use when the user invokes /LV:summary, /summary, asks for a conversation summary, session handoff, or context to pass to another agent. Also /TSM:summary.
disable-model-invocation: true
---

# Conversation Summary (Agent Handoff)

When invoked, produce a **single handoff block** the user can paste as context to another agent. Do not implement new work — summarize only.

## Workflow

1. **Scan the full conversation** — user messages, agent actions, tool results, open questions.
2. **Extract facts from the codebase** when changes are mentioned but details are unclear (files touched, APIs discussed).
3. **Write the handoff** using the template below.
4. **Deliver one fenced block** labeled `agent-context` — nothing else before it except one short intro sentence.

## Output rules

- Write the handoff in the **same language as the conversation** (French if the user spoke French).
- Be **factual and concise** — no filler, no re-litigating options that were rejected unless useful as context.
- Separate **decided** from **open** — never present a pending choice as settled.
- List **concrete file paths** and symbols when code was discussed or changed.
- If nothing was decided or changed, say so explicitly — do not invent progress.
- Keep the handoff under ~800 words unless the session was unusually large.

## Handoff template

Wrap the filled template in a single markdown code fence tagged `agent-context`:

```agent-context
# Agent Handoff — [short session title]

## Goal
[One sentence: what the user is trying to accomplish]

## Context
[2–4 sentences: project, relevant files, constraints mentioned]

## Decisions taken
- **[Decision]** — [rationale or trade-off, if known]
- ...

## Changes already made
- `[path/to/file]` — [what changed]
- ...

_(Write "None" if no code or config was changed.)_

## Changes still required
- [ ] [Concrete next step — file, function, or behavior to implement]
- ...

## Open questions
- [Unresolved choice or missing info]
- ...

_(Write "None" if everything is settled.)_

## Recommended next action
[Single clearest next step for the receiving agent]

## Do not
- [Anti-patterns, rejected approaches, or scope to avoid]
```

## Quality checklist

Before delivering, verify:

- [ ] Every decision reflects what the **user** chose, not assumptions
- [ ] Pending work is actionable (another agent can start without re-reading the full chat)
- [ ] Rejected options are captured in **Do not** when they prevent rework
- [ ] The block is self-contained — no "as discussed above" references
