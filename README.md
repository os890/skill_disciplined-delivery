# Disciplined Delivery — how to use this skill

A reusable Claude Code skill that enforces one engineering discipline across projects: nothing gets implemented that isn't defined, nothing ships without a green build, and no decision is made invisibly.

## Install

Copy the whole `disciplined-delivery/` directory to one of:

| Location | Scope |
| --- | --- |
| `~/.claude/skills/disciplined-delivery/` | All your projects |
| `<project>/.claude/skills/disciplined-delivery/` | That project only, shared with the team via the repo |

The directory name must stay `disciplined-delivery` — it has to match the `name` in `SKILL.md`.

Verify with `/disciplined-delivery` in Claude Code, or just start a work item: the skill triggers on "implement", "add feature", "fix this bug/finding" and "start this ticket".

## Files

| File | Role |
| --- | --- |
| `SKILL.md` | The flow. Always loaded. Ten phases, the scope gate, tool defaults, hard stops. |
| `references/project-setup.md` | Scaffolding, build quality gates, retrofit and init sequences, license compatibility. |
| `references/code-and-tests.md` | Code, comment, DDD, API-contract and test standards. Coverage ratchet. |
| `references/documentation.md` | arc42, Diátaxis, diagrams, screenshots, changelog, visual assets. |
| `references/review.md` | The review brief, handed to the review agent verbatim. |
| `references/release-and-automation.md` | Cutting a release, and turning repeated manual steps into portable scripts. |

The reference files are read on demand, at the phase that needs them.

## Starting a new project

Ask for the project to be initialized, and expect these questions first — the skill will not guess any of them:

1. **License** — required, and never chosen for you.
2. Build tool, target platform version, ticket system, CI system.
3. Which arc42 sections the project will use (full set for an application, a reduced set for a library or CLI).
4. **Accessibility and security requirements** — any target level, regulation, standard or customer obligation. Whatever you name becomes an NFR with measurable criteria, and only then is tooling chosen for it. "No special requirements" is recorded as an explicit answer rather than left silent.

It then creates, in this order:

1. `README.md`, `LICENSE`, `CHANGELOG.md`, `.editorconfig`, `.gitattributes`, `docs/` skeleton.
2. The build file with quality gates wired in from the start — cheapest moment there is — plus hooks and `commitlint`.
3. `ADR-0001` recording the stack, the gate set and the chosen arc42 scope.
4. `docs/requirements/nfr.md`, and the arc42 sections that already have content.
5. CI running the same build a developer runs.
6. A walking skeleton — one thin end-to-end path with a test — before any feature work, so the test infrastructure is proven rather than assumed.

## Adopting it in an existing project

Nothing is switched on silently. The skill reports what is missing and proposes tickets. Gates arrive **one per ticket**, never mixed with a behaviour change, in this order:

1. `.editorconfig`, `.gitattributes`, formatter (large diff, zero behaviour change).
2. `gitleaks` over the working tree **and history** — findings here are rotate-or-rewrite decisions, not backlog items.
3. Enforcer / dependency hygiene.
4. Coverage measurement, gate set to the *current* number and ratcheted upward from there.
5. Static analysis, baselined for existing findings, zero tolerance for new ones.
6. Architecture tests encoding boundaries the code already respects.
7. `commitlint` and `CHANGELOG.md`, starting from the current version — history is not backfilled.
8. Dependency and license scanning, SBOM, then the recurring dependency-update ticket.

## Across sessions

A session starts with no memory of the project, so:

- **It orients first** — project instructions, ADR titles (in full for any the topic touches), the NFRs, arc42 §1/§3/§5, and the build file's gates and thresholds. This is what makes "reuse the existing ADRs" real instead of aspirational, and it will tell you when an existing ADR already decides part of your topic.
- **Continuing a topic is reconstructed, not remembered.** The live task list does not survive a session and is never the record. State is rebuilt from the branch, the ticket, the plan file's step checkboxes, `git log`, the PR, and — decisively — a build run, because what the suite reports beats what the plan claims. You then get the reconstructed state to confirm before any implementation resumes.
- **Anything that must outlive a session is written where it is durable:** confirmed understanding and the scope-gate decision on the ticket, step progress in `docs/plans/<ticket>.md`, decisions in an ADR, blockers as a ticket comment.

## Working an item

**Every new topic opens with a short summary of what was understood** — the goal, who it is for, the assumed current state, what is in and out of scope, and the visible unknowns — and waits for you to confirm or correct it before any requirement is written. It is a mirror, not a proposal: no design and no tool choices in it. This exists because a wrong mental model is cheap to fix at that moment and expensive once it sits in the ticket, the criteria, an ADR, the plan and the tests. It applies to work items, brainstorming and spike requests alike.

Then ten phases, tracked as a live task list so progress is visible:

`1` definition + ticket → `2` clarification until nothing is open → `3` acceptance criteria / ADR / NFR → `4` plan committed and verified against the definitions → `5` TDD → `6` green build incl. all gates → `7` commit + PR → `8` review → `9` documentation → `10` production-readiness.

For genuinely small work there is a **reduced path**: ticket with a one-paragraph definition → TDD → green build → PR, dropping phases 3, 4, 9, 10 and the conformance half of 8. The ticket is never skipped. It is proposed with the skipped phases named and waits for your confirmation — skipping silently would itself be an invisible decision.

A **bug fix** is handled explicitly: it changes behaviour by definition, so it qualifies for the reduced path only when the correct behaviour is already unambiguous from an existing criterion, contract or ADR — reproduce with a failing test, fix, note it in the changelog. If the correct behaviour is *not* defined anywhere, the bug is a definition gap and takes the full flow.

### Brainstorming and quick PoCs

Discussing a new part does **not** start an implementation. A brainstorming session changes no files and creates no branch; it produces options and trade-offs, open questions, candidate ADRs and NFRs, and a draft work item. The flow then starts at phase 1 — the clarification loop still runs, because an idea that felt settled in conversation is not a definition.

Asking for a fast PoC or spike is allowed, and you will be asked to confirm it explicitly first: it suspends the flow (no ticket-backed definition, no TDD, no coverage gate) for **one** `spike/<topic>` branch, with a scope and time box. That permission does not carry to the next request. Spike code is throwaway and never reaches the mainline — the real implementation restarts at phase 1 — and what was learned gets written down before the branch is abandoned, since the knowledge is the actual deliverable. Secrets, licensing and production credentials stay off-limits regardless.

### What it will stop for

- An open question from the clarification phase.
- A decision with no definition behind it.
- **An existing test that would have to change** — you get the reason and the options first.
- A plan that disagrees with the definitions.
- A red suite or a failing gate.
- A phase that would be skipped without confirmation.
- A license that needs choosing.

Stopping means the blocker is stated and everything independent of the answer continues — not that work halts.

## Ground rules worth knowing before you start

- **Coverage**: ~90 % on new and changed code, 80 % floor. In a repo already below it, the gate is set to current coverage and ratcheted; it never decreases.
- **Tests are not edited to make new code pass** without telling you why first. That includes the quiet variants — deleting an assertion, loosening a matcher, `@Disabled`, widening a tolerance, excluding a class from coverage.
- **Gates are never silenced to get green.** Fixing the code is the only route; weakening a gate needs an ADR.
- **Migrations are forward-only**, with reversals written as ordinary migrations, because Flyway's `undo` is a paid feature.
- **Every tool is open source and locally runnable** with no subscription — GitHub as the single accepted exception.
- **Merge commits, no squash** unless you ask.
- **Releases run through the build tool's release mechanism** from a clean tree on a green mainline, with the changelog cut and the SBOM attached. Nothing hand-typed.
- **Anything done by hand twice becomes a committed script** in `scripts/` — POSIX, portable to Linux and containers, `shellcheck`-gated, called by CI and runnable locally.

## Tools and standards

Everything the skill can reach for, why, and which option wins when there are several. All of it is open source, free for commercial use, and runnable on a developer machine natively or in a Podman container — GitHub being the one accepted exception.

| Area | Tool / standard | Why it is used | Role |
| --- | --- | --- | --- |
| Version control | **Git** | History as small coherent commits; merge commits, no squash | Default |
| Remote + tickets | **GitHub** (`gh`, GitHub Issues) | Remote repository, PRs, ticket system | Default — the accepted hosted exception |
| Ticket fallback | `docs/tickets/<id>.md` | Keeps work unblocked when no ticket system is reachable | Fallback only |
| Commit format | **Conventional Commits** | Machine-readable history, feeds changelog generation | Default |
| Commit gate | **commitlint** | Fails the build on a non-conforming commit message | Default |
| Versioning | **SemVer** | Version numbers that mean something to consumers | Default |
| Changelog | **Keep a Changelog** | User-facing record of what changed per release | Default |
| Changelog drafting | **git-cliff** | Drafts entries from the commits; curated by hand afterwards | Default |
| Release automation | `semantic-release` and similar | Automated versioned releases | **Not a default** — requires squashed history; only on explicit request |
| Secret scanning | **gitleaks** (CLI or container) | Keeps credentials out of the diff and the history | Default — never the vendor's CI wrapper action, which is separately licensed |
| Local hooks | **pre-commit** | Fast local mirror of the build's gates | Default (CLI; the hosted service is not needed) |
| Build hygiene | **maven-enforcer-plugin** | Pins tool versions, converges dependencies, bans duplicates and snapshots | Default (Maven); Gradle version catalogs / locking, `npm ci`, `cargo deny` elsewhere |
| Coverage | **JaCoCo** with `check` bound to `verify` | Coverage measured *and* enforced, not just reported | Default (Maven); `cargo-llvm-cov`, `go test -coverprofile`, `coverage.py`, `c8`/`nyc`, Coverlet elsewhere |
| Formatting | The language's own formatter | One canonical format, zero style debate | Default: Spotless (Java), `ruff` (Python), `gofmt` (Go), `rustfmt` (Rust), Prettier (JS/TS), `dotnet format` (.NET) |
| Style checks | **Checkstyle** | Conventions the formatter cannot express | Optional, where the project already uses it |
| Static analysis | **SpotBugs**, **Error Prone**, **NullAway** | Bug patterns and nullability caught at build time | Default (Java); `golangci-lint`, `clippy`, `mypy`/`pyright`, `tsc --noEmit`, Roslyn analyzers elsewhere |
| Architecture rules | **ArchUnit** | Makes bounded contexts and layering fail the build, not just review | Default (Java); `import-linter`, `eslint-plugin-boundaries`, `deptrac` elsewhere |
| Vulnerability scan | **OWASP dependency-check** | Known CVEs in dependencies | Default (Maven; NVD key optional and free) |
| Container/image scan | **Trivy** | Vulnerabilities in images and filesystems | Alternative / complementary |
| SBOM | **CycloneDX** | Answers "what is in this artifact" after release | Default; **Syft** as an alternative generator |
| License headers | **apache-rat-plugin** | Enforces `LICENSE`/`NOTICE` and per-file headers | Default (Maven); `reuse lint`, `cargo deny check licenses`, `license-checker` fork elsewhere |
| Migrations | **Flyway Community** | Versioned, repeatable schema changes, forward-only | Default — `undo` is paid, so reversals are written as ordinary migrations |
| Observability | **OpenTelemetry** + structured JSON logging | Traces and metrics on new paths; NFRs become measurable | Default |
| API contracts | **OpenAPI** (REST), **AsyncAPI** (messaging) | Contract agreed before implementation | Default |
| Contract linting | **Spectral** | Style and completeness of the contract, enforced | Default |
| Breaking-change gate | **oasdiff** (APIs), **japicmp** / **revapi** (Java libs) | An unintended break fails the build; SemVer becomes a fact | Default; `cargo-semver-checks`, `api-extractor` elsewhere |
| Containers | **Podman** | Rootless containers for tests and tooling | Default; **Docker Engine/CLI** as fallback |
| Containers (alt) | Docker Desktop | — | **Not a default** — proprietary, paid above a company-size threshold |
| Java platform | **Jakarta EE APIs + MicroProfile** | Programme against the spec; implementations stay replaceable | Default — needs no permission, unlike any other library |
| Java configuration | **MicroProfile Config** | One standard configuration source instead of a hand-rolled loader | Default |
| Code structure | **Modules along DDD boundaries** | A deployment monolith still gets a modular codebase; boundary violations become compile errors | Default (Maven/Gradle multi-module + ArchUnit) |
| Java integration tests (CDI) | **jawelte** | Bootstraps CDI in-process as a JUnit 6 extension — integration tests written like unit tests, no server, no container | Default for Jakarta/CDI projects |
| Test containers | **Testcontainers** | The suite owns the lifecycle of real dependencies | Default *only* where a real engine cannot run in-process (needs `DOCKER_HOST` + Ryuk disabled under rootless Podman) |
| HTTP fakes | **WireMock** | A real fake server instead of a hand-written mock, at the system edge | Default |
| Contract tests | **Pact** + self-hosted **Pact Broker** | Stubs derived from a verified contract, so services cannot drift | Default where services are owned by different teams |
| Async assertions | **Awaitility** | Awaits a condition instead of sleeping; kills flakiness | Default |
| Property-based tests | **jqwik** / **Hypothesis** / **proptest** | Finds the edge case nobody listed | Encouraged, per language |
| Mutation testing | **pitest** / **Stryker** / **cargo-mutants** | Measures assertion strength, unlike coverage | Optional |
| UI end-to-end tests | **Playwright** | Real browser flows as regression net | Default |
| Screenshots | **Playwright** scripts | Documentation screenshots regenerate instead of rotting | Default |
| Technical doc structure | **arc42** | One predictable place per architecture question | Default (CC BY-SA 4.0; filled-in content stays yours) |
| User doc structure | **Diátaxis** | Tutorial / how-to / reference / explanation, never mixed | Default |
| ADR template | **MADR** | One decision per file, with the rejected options | Default |
| Architecture notation | **C4 model** | Consistent context and building-block views | Default |
| Diagrams | **Mermaid** | Diagram source diffs and reviews like code; renders on GitHub | Default |
| Diagrams (alt) | PlantUML | Richer notation | **Not a default** — GitHub does not render it; GPL by default |
| Wireframes / mockups | **Excalidraw** (`.excalidraw.svg`) | Draft UI that renders on GitHub and stays editable | Default |
| Mockups (alt) | **draw.io** desktop (`.drawio.svg`) | Shape-library mockups, same dual-format trick | Alternative |
| High-fidelity design | **Penpot** (self-hosted) | Design work beyond a wireframe | Alternative |
| Doc gates | **markdownlint**, **lychee** | Broken structure and dead links fail the build | Default |
| Editor consistency | **EditorConfig**, `.gitattributes` | No whitespace or line-ending churn across machines | Default |
| Dependency updates | A recurring ticket | Updates are scheduled work, gated by the suite and the scanner | Default — automation only if the platform already provides it |
| Releases | **`maven-release-plugin`** | Version bump, tag and next development version as one reproducible operation | Default (Maven); `cargo release`, `npm version`, a release plugin or one committed script elsewhere |
| Repeated manual steps | **POSIX scripts in `scripts/`** | Automation instead of re-doing steps by hand each session; portable to Linux and containers | Default |
| Script linting | **shellcheck** | Scripts are code and get a build gate like code | Default |
