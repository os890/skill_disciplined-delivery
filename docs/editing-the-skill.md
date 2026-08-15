# Editing the skill

Rules for changing `SKILL.md` and the files under `references/`.

**This file is not part of the skill.** It is never copied into a project, never loaded by a session, and imposes nothing on anyone using the skill. It exists so the next round of edits follows the same rules as the last one.

## The invariant

**An optimization must not change behaviour.** Shorter is worth nothing if the skill stops asking a question, stops blocking an action, or starts doing something it used to refuse. That is the only rule here the rest is built around.

Three things make a regression here unusually expensive:

- **These files are behaviour, not documentation.** A model reads them and acts on them. Editing them is refactoring production code that happens to be written in prose, and the same discipline applies — not the discipline of tidying a README.
- **Nothing fails.** There is no suite, no gate, no compiler. A deleted condition produces a skill that still loads, still reads fluently, and quietly stops doing one thing. The absence of an error is not evidence.
- **It propagates, and it surfaces late.** Every project that pulls the newer version inherits the regression, and it shows up weeks later as "why did it stop asking about the licence" or "why did it start committing straight to main". By then several projects are affected and the change that caused it is buried in a batch of edits that all looked like cleanup.

The cost is asymmetric, so treat it that way. Keeping forty unnecessary words costs a rounding error in context. Dropping one load-bearing clause costs wrong behaviour in every project that updates, plus the hours spent finding it. **When in doubt, do not cut.**

## What counts as behaviour

Cutting these changes what the skill does, even when the sentence reads like padding:

- **A number or threshold** — 80 %, ~90 %, "2–4 at a time", "one decision per file".
- **A modal** — never, always, unless, only, before, after. These carry the whole rule; the surrounding sentence is scaffolding.
- **A trigger** — the situation that makes a reference get read, or a phase get entered.
- **A named artifact** — the ticket, the plan file, `ADR-0001`, arc42 §11. Replacing one with "record it somewhere" deletes the traceability the skill exists for.
- **An exception or a default** — "off for a project that never answered", "unless the project already uses one".
- **A clause closing a misreading** — "this bounds how many things you take on, never how well you finish the one you took". It looks redundant to a careful reader and is the reason the rule survives a careless one.

Safe to cut: connective prose, a second example that adds no new case, a restatement of framing already established, and an explanation of a point made twice because two edits each set it up.

**Rationale is not padding.** A sentence saying *why* a rule exists is what makes it followed rather than skipped — a model behaves differently given "reuse the ADRs" than given "reuse the ADRs, because one that already decides part of your topic is the cheapest finding available". Cut restatement, keep reasons.

## Every cut needs a stated reason

Before removing anything, name which of these it is:

1. **Restatement** — the same point is made elsewhere *on a path this workflow also reaches*.
2. **Contradiction** — it disagrees with a rule that is staying.
3. **Dead** — it refers to something that no longer exists.

"It feels verbose" is not on the list. If none of the three applies, the text stays, however long the section is.

For (1), **check the reading paths before cutting either copy**. Two locations are often not reached by the same workflows: ask which triggers lead a session to each. If a rule has to hold in a workflow that never opens the other file, the duplication is load-bearing and both copies stay. The spike and brainstorming triggers sit in `SKILL.md` although the detail lives in `references/exploration.md`, precisely because a session has to stop before it has any reason to open that file.

Where duplication is deliberate, make it **asymmetric**: the full statement in one place, a short trigger pointing at it in the other. Two full copies drift apart and then contradict each other; a trigger and its target cannot disagree without it being obvious.

## The check that replaces a test suite

Since nothing fails, the check has to be manual and explicit:

1. **Before editing, list the rules in the section** — one line each, as behaviour: *"asks the licence question"*, *"blocks a spike without per-branch permission"*, *"records the answer in ADR-0001"*.
2. Edit.
3. **Walk the list against the result.** Every entry must still be findable, and still reachable from the same trigger. An entry that moved to another file must have a trigger pointing at it from where the old one was.

If that list is tedious to write, the section is doing more than you thought — which is itself the answer to whether it should be cut.

**Optimizations go in their own commit**, never mixed with a rule change. Same reason the skill itself forbids a gate and a behaviour change in one commit: when a regression turns up three projects later, a pure-cleanup commit can be reverted without losing the rules that were added beside it. Put the before/after word count in the message so the trade is visible.

## Where content goes

**`SKILL.md` is loaded on every session; everything under `references/` is read on demand.** That single fact decides most placement questions.

- A rule that must fire **without anyone deciding to look it up** belongs in `SKILL.md`: the axioms, the hard stops, the phases, and anything that prevents an action rather than shaping one already under way.
- Everything else belongs in a reference — and then **needs a trigger**. Moving detail out without adding a row to the reference-routing table does not save context, it deletes the rule.
- A rule that stops work also needs a line in **Hard stops**. The routing table says where to read; the hard stop says when to stop.
- **`SKILL.md` names no tools.** The test is mechanical: pruning the skill to one stack should touch zero lines of it. If a stack prune wants to edit `SKILL.md`, a tool name has leaked in and belongs in `references/tool-defaults.md`.

## Adding a rule

- **Reconcile it with what is already there.** A new rule that contradicts an old one leaves both weakened. "Commit early, even untested" collided with "keep the suite green at every commit" until green was re-scoped to what gets *pushed*. Search for the rule you are about to contradict before writing the new one.
- **Guard the obvious misreading, in the rule itself.** Every rule has a lazy reading that inverts it: task-only mode reads as "do less of the task", pruning reads as "stop writing decisions down", a threshold for ADRs reads as "skip the ADR". Name the misreading and close it where the rule is stated, not in a note further down.
- **Check the opposite failure is covered.** Rules arrive one-sided: too much duplication but not too little implementation, scope creep but not the unfinished change.
- **Give a test that does not depend on recognising the shape.** "Delete it and run the suite." "What else would have to change if this were reversed next month?" A checkable question survives a case nobody anticipated; a list of examples does not.
- **Scope a check to the diff** where a repository-wide version would be skipped for being slow. A check that runs beats a thorough one that does not.
- **Prefer omission to an empty ritual.** A block that says "none" every time trains the reader to skip it on the day it says something.
- **Make it optional when it is a matter of taste, and record the default.** The work-in-progress commit convention is asked at init and recorded in `ADR-0001`, off for a project that never answered. An opinion imposed by default is how a skill becomes something people work around.

## Conditionality debt

Making one rule conditional creates work everywhere the old rule was stated flatly. When `commitlint` became conditional, seven other places still asserted it — the scaffolding table, the hooks section, both init sequences, the retrofit order, the README twice. Each would have told a project to wire a tool the new rule had just removed.

After making anything conditional, grep for the term and exclude the conditional phrasings:

```sh
grep -rn "commitlint" SKILL.md README.md references/ | grep -viE "unless|not used|is dropped|the default"
```

Whatever remains is either a genuine exception or a contradiction.

## Verify, do not recall

- **Check tool facts before writing them.** Which runner a framework defaults to, whether a platform supports a feature, what a licence file contains — these change, and a defaults file that is confidently wrong is worse than one that is vague.
- **Look at rendered output before committing it.** A diagram that parses is not a diagram that reads; layout defects only appear in the image.
- **Re-check counts you assert.** "Nine diagrams" stayed in the README long after there were eleven.

## After every edit

1. Walk the **before/after rule list** for every section touched.
2. `grep` for contradictions the change introduced — the conditionality check above, and any term whose meaning shifted.
3. If you deleted one of two copies of a rule, confirm the survivor is reachable from **every** workflow that needs it, not just the one you had in mind.
4. Diff each project copy against the skill and confirm **only the intended files differ**. `SKILL.md` should read `0`.
5. Re-read the changed section once as a whole. Edits made one at a time drift, exactly as review fixes do.

## Traps met the hard way

- **Never blind-copy a file into a project copy.** `SKILL.md`, `documentation.md`, `exploration.md` and `review.md` are stack-free and can be copied wholesale. `code-and-tests.md`, `project-setup.md`, `tool-defaults.md` and `release-and-automation.md` are pruned per project — copying the original over one silently discards the prune, and the giveaway is a divergence count dropping to zero.
- **Do not edit Markdown by line index.** Deleting a computed range removed a rule that happened to sit one line further down. Replace unique strings, and assert each one was found.
- **Pinning a stack exposes rules that were right generically.** "Never verify migrations against an embedded database" inverts for a project whose production engine is that database. Read what you pin.
- **A section that grew by accretion is worth restructuring, not trimming.** Three edits that each re-established the framing become one structure — a numbered set of questions, a routing table — with each point stated once. That removes words without removing rules, which trimming rarely does.
