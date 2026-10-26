---
name: "LV:interviewing"
description: Interview the user relentlessly about a plan, decision, or idea until shared understanding. Use when the user wants to stress-test their thinking, or uses LV:interviewing, LV:interview-me, TSM:interviewing, TSM:interview-me, interview-me, interviewing, or /interview-me.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled — the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the frontier in one round, **at most 4 questions**. If the frontier has more than 4, ask the 4 that unblock the most downstream decisions; the rest wait for the next round. Then wait for the user's answers before the next round.

Use the **AskUserQuestion** tool for every frontier question. One call is one round. Do not list options in chat markdown. Each question:

- is multiple-choice, with 2–4 options
- has a short header (max 12 characters)
- puts your recommended option **first**, with `(Recommended)` at the end of the label
- gives each option a label and a one-line description
- is written in the **user's language** (the language they are prompting in)

AskUserQuestion always offers Other — do not add Other yourself. If the user picks Other, wait for their free-text answer before continuing.

Force open-ended decisions into 2–4 plausible frames plus the built-in Other. Never use a markdown Q/A format when AskUserQuestion is available. If AskUserQuestion is unavailable, ask the same questions conversationally with numbered options and a recommended answer.

Each round the user answers reshapes the tree — settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), dispatch a sub-agent to find it — don't ask the user for anything you could look up yourself. Don't block on it: a running exploration is an unsettled prerequisite, so only the questions downstream of it wait for the sub-agent to report — ask the rest of the frontier now. The _decisions_ are the user's — put each to them and wait.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding. When the user confirms you have reached a shared understanding, call EnterPlanMode and write the plan in plan mode. Call ExitPlanMode when the plan is ready. Do not implement until the user approves that plan.
