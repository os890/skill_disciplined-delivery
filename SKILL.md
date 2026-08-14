---
name: disciplined-delivery
description: Project-agnostic engineering discipline and delivery workflow for any work item — feature request, bug, finding, TODO, refactoring — in any language or framework. Enforces definition-before-code (ticket, acceptance criteria, ADR, NFR), a clarification loop with zero assumptions, a committed step-by-step plan verified against the definitions, TDD with a green test suite and build quality gates as the gate, DDD-informed structure, minimal-but-useful comments, docs with Mermaid diagrams and screenshots, PR plus independent code review, and a production-readiness pass. Trigger on "implement", "add feature", "fix this bug/finding", "start this ticket", or whenever work would otherwise begin without a written definition.
---

# Disciplined Delivery

Quality is the constraint, not the goal. Speed comes from not redoing work.

Two rules override everything else in this skill:

1. **No invisible decisions.** Every decision made while implementing must be traceable to a written definition — acceptance criterion, ADR, NFR, or an explicit answer from the user. If it isn't, stop and get it written.
2. **No assumptions.** When something is unclear, ask. Asking three questions now is cheaper than a wrong implementation.

"Work item" below means whatever triggered the work: feature request, bug, finding, TODO, refactoring, technical task.

## Pushback

**Agreement is not the default answer.** When a request looks wrong — the approach, the scope, the tool, the sequencing, the trade-off — say so *before* carrying it out: a sentence or two naming the concern, the alternative you would pick, and what each costs. A reservation swallowed to stay agreeable does not disappear; it comes back as rework, and later.

This is **not** a hard stop. Push back once, then do what the user decides — in full, not a hedged version of it. When they reaffirm, record the trade-off where it outlives the chat: the ticket, or an ADR when it shapes the architecture. A consciously accepted trade-off is still a decision, so rule 1 applies to it like any other.

## Reference files

This file is the flow. Read a reference when the situation arises — not upfront. The trigger is the situation, not the phase number: several references are needed at more than one point.

| Situation | Read |
| --- | --- |
| **About to introduce a tool the project does not already use** | `tool-defaults.md` — what the project already uses wins without opening it |
| Initializing a project; scaffolding or gates missing | `project-setup.md` |
| Writing an ADR or an NFR (phase 3) | `documentation.md` § Architecture decision records |
| Designing a contract, planning, or writing code and tests (phases 4–5) | `code-and-tests.md` |
| A licence or dependency-scan finding, or a tool that fails the bar | `project-setup.md` § License compatibility, § Tools that do not clear the bar |
| A gate fails and someone wants to weaken it | `project-setup.md` § Rules for the gates |
| Writing a changelog entry (phase 7) or cutting one at release | `documentation.md` § Changelog |
| Running the review (phase 8) | `review.md` — hand this file to the review agent |
| Writing docs, diagrams, screenshots or mockups (phase 9) | `documentation.md` |
| Cutting a release, or about to repeat a manual step | `release-and-automation.md` |
| A dependency update item | `project-setup.md` § Dependency updates |

## Starting a session

A session begins with no memory of the project. Two situations, both cheap:

### Orient first — always

Before mirroring a topic back, read enough of the project to speak about it accurately, and no more than that:

- `CLAUDE.md` and any project instructions — they outrank this skill.
- The **ADR titles** in `docs/adr/`, and in full any ADR the topic touches.
- `docs/requirements/nfr.md`.
- arc42 §1, §3 and §5 for the shape of the system; more only if the topic needs it.
- The build file: which gates exist, which coverage threshold is set, which module structure is in place.

This is what makes "reuse existing ADRs" and the precedence rule real rather than aspirational. State briefly what you found that constrains the topic — an existing ADR that already decides part of it is the cheapest possible finding.

### Continuing a topic — reconstruct, then confirm

The task list from an earlier session is **gone**; it is a live view, not a record. Durable state lives in the repository and the ticket. Rebuild from it, in this order:

1. The **branch** — it names the ticket.
2. The **ticket** — definition, acceptance criteria, the confirmed shared-understanding summary, and whether a reduced path was agreed.
3. **`docs/plans/<ticket>.md`** — the plan with its step checkboxes, which is the authoritative record of what is done.
4. **`git log`** on the branch, and the PR state if one exists.
5. **Run the build.** What the suite and the gates actually report beats what the plan claims. A plan step marked done with a red suite means the truth is red.

Then recreate the task list from the plan, and **report the reconstructed state before continuing**: which phase is current, what is done, what the build says, and any open question that was left blocked. Do not resume implementing until the user has seen that summary — a resumption built on a wrong assumption is as expensive as a wrong definition.

## Shared understanding first

**On every new topic, before a single requirement is written or refined, mirror the topic back to the user and wait.** This is the first response on any new topic — a work item, a brainstorming session or a spike request alike.

Keep it short — a handful of lines, not an analysis:

- What you understand the goal to be, in your own words.
- Who it is for, and what problem they have today.
- What you assume the current state to be.
- What you take to be in scope, and explicitly out of scope.
- The risks and unknowns you already see.

Then ask the user to confirm or correct it, and do not move on to definitions until they answer. A wrong mental model is cheapest to fix in this moment: once it has been baked into the ticket, the acceptance criteria, an ADR, the plan and the tests, correcting it means redoing all five.

Two things this is not: it is not a solution proposal — no design, no tool choice, no implementation sketch — and it is not a substitute for the clarification loop in phase 2. It only establishes that both sides are talking about the same thing. If the correction changes the overall picture, mirror it back once more; if it is a detail, note it and continue.

Once confirmed, the summary is not left in the chat: it becomes the opening of the ticket's description in phase 1, so the source of truth is the ticket rather than the conversation.

**Put the scope-gate proposal below in the same message** when the item looks small, and the spike question when a PoC was requested — one round trip, not three.

## Scope gate — how much of this flow applies

The full flow is the default. A **reduced path** exists for genuinely small work, but is never taken silently:

> Name the item as small, list exactly which phases you would skip and why, and wait for confirmation before starting.

Record the confirmed decision on the ticket, so a later session knows which path this item is on.

Reduced path = ticket with a one-paragraph definition → TDD → green build → commit and PR. **The ticket is never skipped** — it is cheap, and it is what makes the work traceable. What the reduced path drops is phases 3, 4, 9 and 10, and the conformance half of phase 8.

It is only a candidate when *all* of these hold: no behaviour change visible to a user or a caller, no public API or contract change, no schema or migration, no *new* dependency (updating an existing one is fine), and existing tests already cover the area.

**A bug fix is a special case.** It changes behaviour by definition, so it cannot meet the first condition — but it does not need the full flow when the correct behaviour is already unambiguous from an existing definition (an acceptance criterion, a contract, an ADR). Then: reproduce with a failing test first, fix, and note the fix in the changelog. If the correct behaviour is *not* already defined, the bug is a definition gap and takes the full flow — that is the interesting case, and the one most often rushed.

Anything else — including "it's just a small feature" — takes the full flow. Skipping a phase without confirmation is itself an invisible decision, which rule 1 forbids.

## Exploration: brainstorming and spikes

**A brainstorming session never turns into an implementation by itself.** Thinking about a new part is welcome and cheap; it produces input for phase 1, not code. During one: no files changed, no branch created, no task list of delivery phases.

What it produces instead: the options considered with their trade-offs, the open questions, the decisions that would need an ADR, the quality attributes that would need an NFR, and a draft work item. Close the session by naming what would become a ticket — then the flow starts at phase 1 with those answers in hand, and the clarification loop still runs. An idea that felt settled in discussion is not a definition.

**A fast proof of concept is a suspension of this flow, and needs explicit permission for exactly one branch.** If asked for a quick PoC, spike or "just try whether this works", ask first:

> This would suspend the normal flow — no ticket-backed definition, no TDD, no coverage gate — for the branch `spike/<topic>` only. The code is throwaway; the knowledge is the deliverable. Scope and time box: <…>. Is that OK for this branch?

Once granted, that permission covers **that branch and nothing else**. It does not carry over to the next request, and it never reaches the mainline:

- Work on `spike/<topic>`, never on a ticket branch and never on the default branch.
- Any PR is marked *not for merge*. Spike code is read for reference, never fast-forwarded into the mainline — the real work restarts at phase 1.
- Write down what was learned before the branch is abandoned: that is the whole point. It lands as a ticket, an ADR, or a section in `docs/plans/`.
- Still binding, permission or not: no secrets in code or history, no license violation, no production credentials, no third-party dependency added to the mainline.

## Precedence

When sources of truth disagree: **project ADRs and `CLAUDE.md` > this skill > your own default**. Surface a conflict you notice; never quietly resolve it.

Other skills and tooling may be reused wherever they do not contradict these rules — check first, then use them rather than reimplementing.

## Tool defaults

**A tool the project already uses wins.** `references/tool-defaults.md` decides only what to *introduce* where nothing exists yet — never a reason to replace a working formatter, coverage tool or test framework. Swapping an established one is its own work item with an ADR, not a side effect. Read the list before introducing a tool the project lacks; reaching for your own habit instead is an invisible decision.

- **Automate the second time.** A manual step done twice becomes a committed script under `scripts/` — *unless a tool or an existing script in the project already does it*, which is the first thing to check (`references/release-and-automation.md`).
- **Never install a heavyweight tool on the host.** Run it from its official container image with the workspace mounted — **Podman** first, Docker Engine/CLI as fallback. If neither the tool nor a container runtime is available, stop and ask; never silently skip the screenshots or the end-to-end tests.

## Progress tracking

Track the applicable phases with the harness task list (`TaskCreate` at the start, `TaskUpdate` to `in_progress` and `completed` as they flip) so the user sees live progress. One task per phase, in order, each named after its heading in *Flow* below.

Omit tasks the scope gate removed, and say which and why. When a phase is blocked, leave it `in_progress` and state the blocker in the reply. Don't also re-post Markdown checklists — the task list is the single view *for this session*.

It does not survive the session, so it is never the record. Anything that must outlive the session goes where it is durable: the confirmed understanding and the scope-gate decision into the **ticket**, step progress into the **plan file**, decisions into an **ADR**, blockers into a ticket comment.

## Flow

### 1. Definition and ticket

Create the ticket first; it is the home of the definition. **GitHub Issues** by default, otherwise the project's system. It records: the kind of item, the problem, the affected users or systems, and what "done" looks like. A one-liner from chat is not a definition.

If no ticket system is reachable, ask the user to create it and supply the ID; if they would rather not, fall back to a committed `docs/tickets/<id>.md` and treat that file as the ticket.

Then create a Git branch named after the ticket and do all work there — plan, implementation, tests and documentation land on that one branch. Never commit to the default branch.

### 2. Clarification loop — the no-assumptions gate

Ask interactively (`AskUserQuestion`, batched, 2–4 at a time) until nothing material is open. Cover at least:

- **Scope**: what is explicitly in, what is explicitly out.
- **Edges**: empty, missing, duplicate, concurrent, oversized, unauthorized, failure of a downstream system.
- **Data**: source of truth, validation rules, migration and backfill of existing data.
- **Non-functionals**: expected load, latency budget, availability, security and privacy, observability, i18n, accessibility — checked against the project-level requirements recorded at init, since an individual item can carry an implication those never anticipated.
- **Integration**: which systems, which contracts, versioning and backwards compatibility.
- **Operations**: configuration, feature flag, rollout and rollback.

Do not smooth over a gap with a "reasonable default". A default is a decision, and decisions get written down (phase 3). Refine the ticket with the answers, so the ticket — not the chat log — stays the source of truth.

### 3. Anchor the requirements

| Kind of decision | Where it lives |
| --- | --- |
| Business behaviour | Acceptance criteria on the ticket (Given/When/Then) |
| Architecture / structural choice | ADR under `docs/adr/`, **MADR** format, one decision per file, rejected options recorded, never rewritten once decided (`references/documentation.md`) |
| Quality attribute (perf, security, availability, …) | NFR in `docs/requirements/nfr.md`, with a measurable number |
| An interface others consume | A committed **OpenAPI**/**AsyncAPI** contract, agreed before implementation (`references/code-and-tests.md`) |

Reuse existing ADRs and NFRs; write a new one only when the decision isn't covered. Each acceptance criterion must be testable — if you cannot see the test, the criterion is not finished.

### 4. Plan

Write the step-by-step plan to `docs/plans/<ticket>.md` and commit it to the working branch **before** implementing.

Per step, as a Markdown checkbox: what changes, which acceptance criterion or NFR it serves, which tests come first, and the expected observable result.

**The plan is the durable progress record.** Tick a step and commit that change as the step completes, so a later session can see where the work stands without reading the whole diff.

Then verify the plan against the definitions and answer explicitly:

- Does every step trace back to a definition? Anything that doesn't is scope creep — remove it or get it defined.
- Is every acceptance criterion covered by at least one step?
- Does any step contradict an ADR or NFR?

Present the verification result to the user before starting.

### 5. Implementation — TDD

Red → green → refactor, per plan step. Commit in small, coherent steps that keep the suite green. Standards for the code, the tests and the structure: `references/code-and-tests.md`.

If you discover something the definitions don't cover, **pause that part**: post the open question and mark the phase blocked — but continue with every step that does not depend on the answer, and batch open questions into one ask rather than stopping the whole task per question. Never invent the answer and carry on.

### 6. Build gate

The task is not finished while a test fails. Run the project's full build — compile, tests, and every quality gate — and report the actual output, including failures.

A failing gate is a finding, not an obstacle: fix the code. **Never disable a plugin, exclude a class, lower a threshold or add a suppression to get green** without an ADR and the user's agreement.

### 7. Commit and PR

- Commit subject follows **Conventional Commits** and carries the ticket number when the system has one: `feat(billing): #42 round settlement amounts half-up`.
- Every commit you author carries a `Co-Authored-By:` trailer naming the model.
- A user-visible change adds a `CHANGELOG.md` entry under *Unreleased*, written from the user's point of view — not a paste of the commit subject — grouped as Added / Changed / Deprecated / Removed / Fixed / Security, with a migration note for a breaking change (`references/documentation.md`).
- Push the branch and open a PR where the project supports them.
- **Merge commits, no squash** unless the user asks — the small coherent commits are the history.
- The PR description links the ticket, summarises the change, notes deviations from the plan and why, and lists what a reviewer should look at.

### 8. Review

While the user reviews, review independently. Findings are reported, never silently fixed.

Reuse the project's existing review tooling for the mechanical pass where it does not contradict this skill — a `/code-review` skill for correctness and quality, a `/security-review` skill where the change touches authentication, authorization, data handling or an external boundary — and add a sub-agent for what generic tooling cannot know: **conformance** to the plan, the acceptance criteria, the ADRs and the NFRs. Give that agent all four, plus `references/review.md` as its brief — without them it cannot check conformance at all.

"No findings" is a valid, useful result — report it plainly rather than manufacturing filler. Triage every real finding with the user: fix now, ticket it, or reject with a reason.

### 9. Documentation

After the code is settled, check documentation for both missing and outdated content — see `references/documentation.md`.

- **End-user documentation** (`docs/user/`, structured per **Diátaxis**) for every user-visible change, with screenshots of the real build, or a clearly-marked mockup where the UI does not exist yet. A purely technical item may skip it; say so explicitly.
- **`CHANGELOG.md`** — confirm the phase 7 entry still matches what actually shipped.
- **Technical documentation** always, always with Mermaid diagrams chosen to fit the subject. Structure it as **arc42** under `docs/technical/arc42/` — updating the affected sections of the system's document, not creating a new one per item. §9 indexes the ADRs and §10 the NFRs by link, never by copy.

### 10. Production-readiness review

A final pass over the whole change: configuration and secrets, migrations and their tested forward reversal, logging and metrics for the new paths, failure modes and timeouts, security and authorization, performance against the NFR, feature flag and rollout, leftover TODOs, debug code, hardcoded values. Report anything not production ready rather than shipping it quietly.

Anything the user consciously accepts instead of fixing gets recorded in arc42 §11 (Risks and Technical Debt) with a ticket — accepted is not the same as forgotten.

## Hard stops

Stop, say so, and involve the user when:

- A question from phase 2 is still open.
- A decision has no backing definition.
- An existing test would have to change (protocol in `references/code-and-tests.md`).
- The plan and the definitions disagree.
- The suite is red, or a quality gate fails.
- A phase would be skipped without confirmation.
- A license must be chosen — that is the user's decision, never yours.
- A **third-party dependency** would be added that the platform does not already provide.
- A **tool the project does not already use** would be introduced — build plugin, framework, linter, diagram format, container runtime — without `references/tool-defaults.md` having been read first, or an established one would be swapped without an ADR.
- A **spike or PoC** would start without explicit per-branch permission, or spike code would reach the mainline.
- A brainstorming session is about to become an implementation without passing through phase 1.
- The **shared-understanding summary** for a new topic has not been confirmed yet.

Stopping means: state the blocker and what you need, keep working on everything independent of the answer, and never substitute a guess for the answer.
