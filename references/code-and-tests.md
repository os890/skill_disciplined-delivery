# Code, structure and test standards

Read this when planning and implementing (phases 4–5), and when judging structure or coverage.

## Platform and dependencies

**A new third-party dependency is the user's decision, not yours.** Adding a library is an architecture decision with a maintenance, licensing and supply-chain cost attached. Propose it, name what it buys and what it would replace, and wait for permission. Never pull one in because it is convenient.

Two things are already approved and need no separate permission: **the platform surface**, and **the tools this skill names as defaults** (the test framework, the migration tool, the observability API and so on — the user adopted them by adopting this skill). Everything else is a proposal.

What the platform offers, for Java:

- **Jakarta EE APIs first.** Programme against the specification — CDI, JPA, JTA, Bean Validation, JAX-RS, Jakarta Batch — rather than a framework's own abstraction over it. The API is the stable surface; implementations are replaceable.
- **MicroProfile for what Jakarta EE leaves open**, and it counts as part of the platform default in the same way.
- **Configuration via MicroProfile Config.** No hand-rolled properties loader, no bespoke environment-variable parsing, no configuration framework pulled in for it — unless the user asks. A home-grown configuration mechanism is exactly the kind of invisible decision this skill exists to prevent.
- Anything beyond that surface and this skill's own defaults — a utility library, a mapper, a framework — is a proposal, not a default.

## Code standards

- **Efficient and performance-tuned**: right algorithm and data structure first; avoid repeated work, needless copies, chatty I/O, N+1 queries. Tune against the NFR's number, not a hunch — measure before optimizing anything non-obvious, and say what you measured.
- **Readable and maintainable**: intent-revealing names, small units with one responsibility, shallow nesting, explicit over clever. Code that reads like the surrounding code.
- **Formatted with the language's default formatter only** — `gofmt`, `rustfmt`, `dotnet format`, Prettier, Black/Ruff format, Spotless with the project's chosen style. No hand-tuned alignment, no mixed styles, no reformatting untouched code. The formatter check runs in the build.
- **No duplication by copy-paste.** Extract when the second occurrence appears and the abstraction is real; keep both when they merely look alike.
- **No speculative generality.** No configuration flag, extension point or abstraction layer without a requirement behind it — that is scope creep wearing an architecture costume.
- **Fail loudly**: validate at the boundary, no empty catch blocks, no error swallowed into a log line, no `null` returned where absence is a real state to model.
- **Observable by default**: instrument new operations and outbound calls with **OpenTelemetry** spans and metrics, and log in a structured (JSON) format with a correlation identifier. Never log a secret, token or personal datum — the log is a copy of your data with weaker access control.
- **Config from the environment**, never a hardcoded host, path, credential or limit. A value someone will want to change in production belongs in configuration, with a documented default.

## Comments and documentation in code

Short as useful, never text for its own sake. A comment or doc block earns its place by explaining *why*, a constraint, a trade-off, a non-obvious invariant, or a gotcha — never by restating the signature.

```java
// Delete this: adds nothing.
/** Gets the customer id. @return the customer id */

// Keep this: encodes a decision the code cannot show.
/** Uses the settlement date, not the booking date — see ADR-014. */
```

Public API is the exception where completeness matters: document parameters that have constraints, what is thrown and when, thread-safety, and nullability — because callers cannot read the body. Still no filler.

A comment explaining *what* the code does usually means the code should be clearer instead. A comment that repeats an ADR should link the ADR, not paraphrase it.

## Structure — DDD where it pays

Use the DDD elements that improve structure: bounded contexts, ubiquitous language in the code, aggregates with clear invariants and boundaries, value objects for concepts with no identity, domain events, repositories at the aggregate boundary, application services for orchestration.

Apply them pragmatically, not dogmatically. When the language or framework offers a better-fitting modern alternative — records and sealed hierarchies for value objects and states, framework-native persistence instead of a hand-rolled repository, an outbox or streaming mechanism instead of a bespoke event bus — take it, and record the choice in an ADR.

Never optional: a domain model with real boundaries, and a language in the code that matches the business.

### A deployment monolith is not a code monolith

**Split the codebase into modules along DDD lines by default** — one module per bounded context, with the aggregates inside it — even when the whole thing deploys as a single unit. Deploying as one artifact is a runtime decision; structuring as one undifferentiated codebase is a design failure, and the two are unrelated.

- Use the build tool's own module mechanism (Maven or Gradle multi-module, and the language's module system where it has one), so a boundary violation is a **compile error**, not a review comment.
- Each module declares what it exposes; everything else stays internal. A dependency between modules is deliberate, visible in the build file, and reviewable.
- Domain modules do not depend on infrastructure or on delivery mechanisms; the dependency arrows point inward.
- Cross-context communication goes through a published interface or a domain event, never a direct reach into another context's internals.
- Keep the deployment simple as long as the domain allows it — this structure means a context can later become its own deployable without a rewrite, but do not split the deployment on speculation.
- Boundaries are **enforceable, not aspirational**: the build's module graph catches what it can, and ArchUnit tests (or the equivalent import linter) cover the rest — naming, layering inside a module, allowed annotations, a domain class reaching into infrastructure. Either way it fails the build instead of surviving review.

## API contracts

Where the work item touches an interface others consume — REST, messaging, or a published library API — the contract is written **before** the implementation and committed to the repository: **OpenAPI** for REST, **AsyncAPI** for events and messages.

- **Contract-first.** The contract is part of the plan (phase 4) and reviewable on its own; agreeing on it is cheaper than agreeing on an implementation.
- Lint it with **Spectral** in the build, so style and completeness are enforced rather than debated.
- Gate breaking changes: `oasdiff` for OpenAPI, japicmp/revapi for a library. A break that is intended gets a SemVer major, a `CHANGELOG.md` migration note, and an ADR if it affects architecture.
- Generate from the contract where the ecosystem supports it (server stubs, clients, docs) rather than maintaining two truths by hand. Generated artifacts are linked from the docs, never hand-copied.
- Contract tests (**Pact**) verify that the provider actually honours what the consumer relies on — the disciplined alternative to mocking a service you own.

## Testing standards

Tests are the backbone. They exist to catch breakage nobody was looking for.

- **TDD wherever possible**: the failing test comes first. A test that has never been seen to fail proves nothing.
- **Test the real flow whenever possible.** Unit tests cover the basics; a real end-to-end path through the actual components is what proves the feature works.
- **Mocks are a last resort**, acceptable mainly at the edge of your system — a third-party API, a payment provider, a clock. Never mock your own domain to avoid wiring it up. A test suite made of mocks tests the mocks.
  - Where you do stub an external HTTP service, prefer a real fake over a hand-written mock: **WireMock** (or the ecosystem's equivalent) as an actual server — embedded where the test framework manages its lifecycle, as a container otherwise.
  - Between services you own, a hand-written stub is a guess. **Pact** or another consumer-driven contract test makes the stub derive from a contract the provider verifies, so the two cannot drift silently.
- **Property-based tests** for anything with an interesting input space — parsing, rounding, ordering, invariants, serialisation round-trips (jqwik, Hypothesis, proptest). One property replaces a dozen hand-picked cases and finds the edge you did not think of, which is exactly what the phase 2 edge catalogue is for.
- **Embedded first**: in-process test framework, embedded server, embedded broker or database where a credible one exists.
  - **Java with CDI / Jakarta EE → [jawelte](https://github.com/os890/jawelte)** (Apache-2.0, `org.os890.jawelte`) is the default for integration tests. It registers as a JUnit 6 extension and bootstraps a CDI container inside the test JVM, per class or per method, so an integration test is written like an ordinary unit test with no application server and no container runtime. `@EnableTestBeans` on the test class, and beans are injected and lifecycle-managed for you.
  - Its opt-in modules cover the rest of the flow with meaningful defaults instead of configuration: JPA, EJB, JTA, JAX-RS, MicroProfile, Jakarta Batch, H2, DB-Unit for dataset-driven database state, Mockito for CDI-aware mocks, WireMock for HTTP stubs, plus flow assertions. Configuration comes from MicroProfile Config, matching the platform rule above.
  - Persistence tests share one transaction across setup, execution and assertions, rolled back by default — so tests stay isolated without a per-test database reset.
  - This is what "embedded first" looks like in practice: reach for Testcontainers only when the test genuinely needs a real engine that cannot run in-process.
- **Containers when embedded won't do** — the app needs a real runtime, or a real database, message broker or cache. **Podman** by default (Docker Engine/CLI if the project already requires it — not Docker Desktop, which is paid above a company-size threshold); drive it from a framework such as Testcontainers so the suite owns the lifecycle and nothing leaks between runs.
  - Testcontainers on rootless Podman typically needs `DOCKER_HOST` pointed at the Podman socket (`unix://$XDG_RUNTIME_DIR/podman/podman.sock`) and the Ryuk reaper disabled (`TESTCONTAINERS_RYUK_DISABLED=true`), since Ryuk does not work rootless. Put that configuration in the build, not in each developer's shell profile, and verify it against the current Testcontainers documentation — the details shift between versions.
- **Integration and end-to-end suites** belong in the project, run by a framework, kept working over time. That regression net is the point. For a UI that framework is **Playwright** — and the same scripts can produce the documentation screenshots, so the docs stay honest for free.
- **Deterministic**: no dependence on wall-clock time, timezone, locale, network, execution order or `sleep`. Inject the clock; await a condition instead of pausing — **Awaitility** or the equivalent, never a fixed wait. A `sleep` in a test is a race condition with a delay.
- **Migrations are tested, not assumed**: the suite runs **Flyway** from an empty schema *and* from the previous release's schema, so the upgrade path is exercised, not just the end state.
  - This is the one place where "embedded first" does **not** apply: migrations run against **the production database engine in a container**, never against an in-memory substitute. An embedded database accepts SQL the real engine rejects and rejects SQL it accepts, so a green migration test on H2 says nothing about PostgreSQL. Embedded databases stay fine for behaviour tests that do not exercise schema changes.
  - **Forward-only.** Flyway's `undo` is a paid feature, so never design a release around it. Where a way back is needed, write the reversal as an ordinary versioned migration — it is then a migration like any other, and therefore testable and free.
  - Prefer **expand/contract** for anything with live traffic: add the new shape, migrate the data, switch the readers, remove the old shape in a later release. Each step is independently deployable and reversible by going forward.
  - A rollback you claim in the PR is a rollback the suite executes. An untested reversal is a hope.
- **Public API compatibility** for a library: a gate (japicmp, revapi, `cargo-semver-checks`) that fails on an unintended breaking change, so SemVer is a fact rather than an intention.
- **Test behaviour, not implementation.** A test that breaks on every harmless refactoring is a liability; a test that still passes while the code is broken is worthless.
- **One reason to fail per test**, and a name that says what behaviour is expected.
- **Every acceptance criterion maps to a test that is recognisably its test** — named after the criterion, or referencing it. That is what makes phase 3's "each criterion must be testable" verifiable in phase 8, and it is the cheap alternative to a separate specification framework: the criterion and the test stay one artifact apart, not one translation layer apart.
- **The suite runs as part of the build**, and a task can only be finished with a passing suite.

### Coverage

Target ~90 % line and branch on new or changed code. 80 % is the floor, not the goal.

In a repository already below the floor, **ratchet** instead of blocking:

1. Set the enforced global gate to the current measured coverage, and report that number.
2. New and changed code meets the target.
3. The gate may never decrease.
4. Lifting it toward 80 % becomes its own ticket.

Coverage is a smoke detector, not a score — high coverage with weak assertions is a failing suite. Where the project wants a real signal on assertion quality, add mutation testing (pitest, Stryker, `cargo-mutants`) instead of chasing percentage points.

### Never change a test silently

Before modifying or deleting an existing test or assertion, tell the user first:

> Test `X` fails with this change. Cause: <the behaviour change>. My reading: <intended change of contract | the new code is wrong>. Proposed adjustment: <what and why>. Alternative: <keep the behaviour instead>.

Then wait. A failing existing test is evidence, and the default assumption is that the new code is wrong — not the test.

The same applies to the quieter versions of the same move: deleting an assertion, loosening a matcher, adding `@Disabled`/`skip`/`ignore`, widening a tolerance, excluding a class from coverage, or catching the exception the test was there to prove. All of these need the same notification.
