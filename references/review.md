# Review brief

This is the brief for phase 8. Hand this file to the review agent verbatim.

## Inputs the reviewer must receive

Without all four, conformance cannot be checked and the review is guesswork:

1. The **diff** under review (or the PR number / branch).
2. The **plan** — `docs/plans/<ticket>.md`.
3. The **ticket** with its acceptance criteria.
4. The relevant **ADRs** and **NFRs**.

## Stance

Report findings; do **not** silently fix them. Be specific: file, line, what is wrong, and the concrete input or state that makes it wrong. A finding nobody can reproduce from your description is not yet a finding.

**"No findings" is a valid and useful result.** Say it plainly. Never manufacture filler to look thorough — a review padded with style opinions trains the reader to skip real findings.

Rank by severity, most severe first. Prefer few confirmed findings over many speculative ones, and mark anything you could not verify as such.

## What to check

### Conformance — the part generic tooling cannot do

- Does the implementation satisfy **every acceptance criterion**, and is each one actually covered by a test?
- Did the implementation follow the **plan**? Deviations are allowed but must be stated in the PR — an unexplained deviation is a finding.
- Does anything **contradict an ADR or an NFR**? Is a performance-related NFR backed by a measurement rather than an assumption?
- Is anything present that **no definition asked for** — an extra flag, an unused extension point, a speculative abstraction? That is scope creep, and it is a finding.
- Is anything **a definition asked for missing**? Only the happy path implemented, an edge case from phase 2 unhandled, another caller of the changed code left untouched, a value hardcoded that an NFR says is configurable, an error path that returns but does not report. This is the harder half of the same check: scope creep is visible in the diff, whereas an unfinished change looks like restraint. Weigh it the same.
- Was a **third-party dependency added** without the user's explicit permission? Was something hand-rolled that the platform already provides — configuration in particular?
- Do the **module boundaries** hold? A new dependency between modules, a domain module reaching into infrastructure, or a context reaching into another context's internals is a finding even when it compiles.
- Was any **decision made invisibly** — a default, a limit, a timeout, a rounding rule, a retry count — without a definition behind it?

### Contracts, history and release artifacts

- Does the commit history follow **Conventional Commits** with the ticket number, and is each commit coherent on its own?
- Is there a **`CHANGELOG.md`** entry for anything a user or integrator notices, written for them rather than pasted from the commit — and a migration note for a breaking change?
- If an **API contract** changed: is the OpenAPI / AsyncAPI file updated in the same PR, is the change non-breaking, and if it breaks, is that reflected in the version, the changelog and an ADR?
- If a **migration** was added: is it exercised from an empty schema *and* from the previous release's schema, and is the claimed rollback actually tested?

### Correctness

- Edge cases: empty, missing, duplicate, boundary, oversized, concurrent, unauthorized, downstream failure.
- Error handling: nothing swallowed, nothing logged-and-continued that should abort, no error path that loses data.
- Concurrency: shared mutable state, race windows, non-atomic read-modify-write, lock ordering.
- Resource lifecycles: closed streams, released connections, cancelled subscriptions, bounded queues and caches.
- Data: migrations reversible, backfill correct, no accidental full-table operation.

### Efficiency

- Needless allocation, repeated computation of the same value, work done inside a loop that belongs outside.
- N+1 access, missing batching, chatty I/O, missing index for a new query pattern.
- Wrong data structure for the access pattern.
- Optimisation without a measurement — as much a finding as a missing optimisation.

### Verbosity and duplication

- Copy-pasted blocks that should be one thing; near-identical branches that differ only in a constant.
- Dead abstraction: an interface with one implementation and no second caller in sight.
- Code that says the same thing twice — a check already guaranteed by the type, a comment restating the line below it.
- Anything that could be deleted with no loss of behaviour.

### Tests

- Do they test **behaviour or implementation detail**? Would a harmless refactoring break them?
- Are they **correct on their own** — right assertion, right expected value, actually asserting something?
- **Would they fail if the code were wrong?** Look for assertions that cannot fail, tests with no assertion, and swallowed exceptions.
- Are the **edge cases from the clarification phase** covered, or only the happy path?
- Are mocks used where a real component was possible? Is any of the user's own domain mocked?
- Determinism: time, timezone, locale, ordering, network, `sleep`.
- Was an **existing test changed or disabled**? If so, was the user informed with a reason? An undisclosed test change is a high-severity finding.

### Comments and documentation

- Does each comment add something the code cannot say, or is it restating the signature?
- Is a non-obvious decision left undocumented, or documented in a comment where it belongs in an ADR?
- Is the user documentation updated for a user-visible change; is the technical documentation and its diagrams still accurate?

### Production readiness

- Secrets or hostnames in code, config or fixtures; anything sensitive in a screenshot or log line. A secret in the diff is the highest-severity finding there is — it outlives the revert.
- Missing observability on the new paths: no span or metric on the new operation, no log at the failure point, no correlation identifier; or the opposite — a token or personal datum written into a log.
- Hardcoded limit, timeout, host or path that belongs in configuration.
- Missing timeout, retry or backoff on an outbound call; unbounded growth.
- Leftover TODO, commented-out code, debug output, hardcoded value that belongs in config.

## Triage

Every real finding gets an outcome, decided with the user: **fix now**, **ticket it**, or **reject with a reason**. Nothing is left in the "mentioned once and forgotten" state.
