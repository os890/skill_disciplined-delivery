# Editing the skill

Rules for changing `SKILL.md` and the files under `references/` — how to add a rule, and how to keep the result from growing into something nobody reads.

**This file is not part of the skill.** It is never copied into a project, never loaded by a session, and imposes nothing on anyone using the skill. It exists so the next round of edits follows the same rules as the last one.

## Where content goes

**`SKILL.md` is loaded on every session; everything under `references/` is read on demand.** That single fact decides most placement questions.

- A rule that must fire **without anyone deciding to look it up** belongs in `SKILL.md`: the axioms, the hard stops, the phases, and anything that prevents an action rather than shaping one already under way.
- Everything else belongs in a reference — and then **needs a trigger**. Moving detail out without adding a row to the reference-routing table does not save context, it deletes the rule.
- A rule that stops work also needs a line in **Hard stops**. The routing table says where to read; the hard stop says when to stop.
- **`SKILL.md` names no tools.** The test is mechanical: pruning the skill to one stack should touch zero lines of it. If a stack prune wants to edit `SKILL.md`, a tool name has leaked in and belongs in `references/tool-defaults.md`.

## What to cut, and what never to cut

- **Cut restatement. Keep rationale.** A sentence explaining *why* a rule exists reads like padding and is the reason the rule gets followed instead of skipped — this file is consumed by a model before every task, and "reuse the ADRs" behaves differently from "reuse the ADRs, because an existing one that already decides part of your topic is the cheapest finding available". Restatement is different: the same point made twice because two edits each re-established the framing.
- **Prefer deleting duplication across files over shortening prose — but check the reading paths before cutting either copy.** The largest reductions came from removing things already said elsewhere. The trap is that two locations are not reached by the same workflows: ask which triggers lead a session to each. If a rule has to hold in a workflow that never opens the other file, the duplication is load-bearing and both copies stay. The spike and brainstorming triggers sit in `SKILL.md` although the detail lives in `references/exploration.md`, precisely because a session has to stop before it has any reason to open that file.
  - Where duplication is deliberate, make it **asymmetric**: the full statement in one place, a short trigger pointing at it in the other. Two full copies drift apart and then contradict each other; a trigger and its target cannot disagree without it being obvious.
- **Accretion is the thing to look for.** Sections edited three times in three sessions usually say the same thing three ways. The fix is one structure — a numbered set of questions, a routing table — with each point stated once and examples carrying the weight.
- **Count before and after, and report it honestly.** Most cuts are worth a few percent. The gain is legibility, not tokens; claiming otherwise invites a bad trade later.

## Adding a rule

- **Reconcile it with what is already there.** A new rule that contradicts an old one leaves both weakened. "Commit early, even untested" collided with "keep the suite green at every commit" until green was re-scoped to what gets *pushed*. Search for the rule you are about to contradict before writing the new one.
- **Guard the obvious misreading, in the rule itself.** Every rule has a lazy reading that inverts it: task-only mode reads as "do less of the task", pruning reads as "stop writing decisions down", a threshold for ADRs reads as "skip the ADR". Name the misreading and close it where the rule is stated, not in a note further down.
- **Check the opposite failure is covered.** Rules arrive one-sided: too much duplication but not too little implementation, scope creep but not the unfinished change. If the new rule has a mirror image, it probably needs stating too.
- **Give a test that does not depend on recognising the shape.** "Delete it and run the suite." "What else would have to change if this were reversed next month?" A checkable question survives contact with a case nobody anticipated; a list of examples does not.
- **Scope a check to the diff** where a repository-wide version would be skipped for being slow. A check that runs is worth more than a thorough one that does not.
- **Prefer omission to an empty ritual.** A block that says "none" every time trains the reader to skip it on the day it says something.
- **Make it optional when it is a matter of taste, and record the default.** The work-in-progress commit convention is asked at init and recorded in `ADR-0001`, and it is off for a project that never answered. An opinion imposed by default is how a skill becomes something people work around.

## Conditionality debt

Making one rule conditional creates work everywhere the old rule was stated flatly. When `commitlint` became conditional, seven other places still asserted it — the scaffolding table, the hooks section, both init sequences, the retrofit order, the README twice.

After making anything conditional, grep for the term and exclude the conditional phrasings:

```sh
grep -rn "commitlint" SKILL.md README.md references/ | grep -viE "unless|not used|is dropped|the default"
```

Whatever remains is either a genuine exception or a contradiction.

## Verify, do not recall

- **Check tool facts before writing them.** Which runner a framework defaults to, whether a platform supports a feature, what a licence file actually contains — these change, and a defaults file that is wrong is worse than one that is vague. Verify, then write, and say what was verified.
- **Look at rendered output before committing it.** A diagram that parses is not a diagram that reads; layout defects only appear in the image.
- **Re-check counts you assert.** "Nine diagrams" stayed in the README long after there were eleven.

## After every edit

1. `grep` for contradictions the change introduced — the conditionality check above, and any term whose meaning shifted.
2. Word-count the sections you touched, before and after.
3. If you deleted one of two copies of a rule, confirm the survivor is reachable from **every** workflow that needs it — not just the one you had in mind.
4. Diff each project copy against the skill and confirm **only the intended files differ**. `SKILL.md` should read `0`.
5. Re-read the changed section once as a whole. Edits made one at a time drift, in the same way review fixes do.

## Traps met the hard way

- **Never blind-copy a file into a project copy.** `SKILL.md`, `documentation.md`, `exploration.md` and `review.md` are stack-free and can be copied wholesale. `code-and-tests.md`, `project-setup.md`, `tool-defaults.md` and `release-and-automation.md` are pruned per project — copying the original over one silently discards the prune, and the giveaway is a divergence count dropping to zero.
- **Do not edit Markdown by line index.** Deleting a computed range removed a rule that happened to sit one line further down. Replace unique strings; assert each one was found.
- **Pinning a stack exposes rules that were right generically.** "Never verify migrations against an embedded database" inverts for a project whose production engine is that database. Read what you pin.
