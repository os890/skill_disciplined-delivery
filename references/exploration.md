# Exploration: brainstorming and spikes

Read this when a topic is being explored rather than delivered — a brainstorming session, a "let's think about X", a request for a quick proof of concept.

## Brainstorming

**A brainstorming session never turns into an implementation by itself.** Thinking about a new part is welcome and cheap; it produces input for phase 1, not code. During one: no files changed, no branch created, no task list of delivery phases.

What it produces instead: the options considered with their trade-offs, the open questions, the decisions that would need an ADR, the quality attributes that would need an NFR, and a draft work item.

Close the session by naming what would become a ticket — then the flow starts at phase 1 with those answers in hand, and the clarification loop still runs. An idea that felt settled in discussion is not a definition.

## Proof of concept / spike

**A fast proof of concept is a suspension of this flow, and needs explicit permission for exactly one branch.** If asked for a quick PoC, spike or "just try whether this works", ask first:

> This would suspend the normal flow — no ticket-backed definition, no TDD, no coverage gate — for the branch `spike/<topic>` only. The code is throwaway; the knowledge is the deliverable. Scope and time box: <…>. Is that OK for this branch?

Once granted, that permission covers **that branch and nothing else**. It does not carry over to the next request, and it never reaches the mainline:

- Work on `spike/<topic>`, never on a ticket branch and never on the default branch.
- Any PR is marked *not for merge*. Spike code is read for reference, never fast-forwarded into the mainline — the real work restarts at phase 1.
- Write down what was learned before the branch is abandoned: that is the whole point. It lands as a ticket, an ADR, or a section in `docs/plans/`.
- Still binding, permission or not: no secrets in code or history, no license violation, no production credentials, no third-party dependency added to the mainline.
