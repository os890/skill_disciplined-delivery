# Disciplined Delivery — how to use this skill

A reusable Claude Code skill that enforces one engineering discipline across projects: nothing gets implemented that isn't defined, nothing ships without a green build, and no decision is made invisibly.

![The whole approach: orient and mirror the topic back, pass the scope gate, then either the reduced path or ten phases from Define to Finish — with four rules running underneath every phase: no invisible decisions, no assumptions, push back once, and nothing durable living in the session.](docs/diagrams/delivery-flow-1.png)

Ten more diagrams break this down — routing, the phases and their artifacts, the decision loop, hard stops, session state, the build gate and the tooling rules: **[docs/diagrams/delivery-flow.md](docs/diagrams/delivery-flow.md)**.

## Install

Copy the whole `disciplined-delivery/` directory to one of:

| Location | Scope |
| --- | --- |
| `~/.claude/skills/disciplined-delivery/` | All your projects |
| `<project>/.claude/skills/disciplined-delivery/` | That project only, shared with the team via the repo |

The directory name must stay `disciplined-delivery` — it has to match the `name` in `SKILL.md`.

Verify with `/disciplined-delivery` in Claude Code, or just start a work item: the skill triggers on "implement", "add feature", "fix this bug/finding" and "start this ticket".

## Trimming it to one stack

The skill is stack-agnostic on purpose: every default names the alternatives — Maven and Gradle, `npm ci` and `cargo deny`, Spotless and `gofmt` and `dotnet format`. That breadth is what makes it reusable, and it costs nothing in the flow itself. It does cost something in the references a project actually opens, where a paragraph about `cargo-semver-checks` is noise for a team that will never write Rust.

So for a project whose stack is settled, keep a **pruned copy** beside the project rather than the general one — and **ask Claude to produce it**, rather than editing seven files by hand.

Do that **from a session in the skill's own repository, pointed at the target project**. Not from inside the target project: a session there only ever sees the pruned copy, which is the whole point of making one — it cannot prune what it can no longer read, and it would be rewriting the rules it is working under.

> Copy this skill to `<path-to-project>/.claude/skills/disciplined-delivery/`, then strip everything in the copy that does not apply to that project's stack: Java 25, Maven, Spring Boot, PostgreSQL, Angular. Pin the defaults to those, and tell me anything the pinning contradicts.

What that should produce, and what to check when it reports back:

1. The directory copied to `<project>/.claude/skills/disciplined-delivery/`, without `README.md` and `docs/` — neither is loaded by the skill, so both are pure weight in a project copy.
2. The alternatives you will never use deleted: the *Elsewhere* column of the build-gates table, and the per-language lists for formatter, property tests, mutation testing and the release mechanism.
3. What is left pinned: `| Formatting | The language's default formatter |` becomes `| Formatting | **Spotless** |`, and the platform row names your actual framework and version.
4. `SKILL.md` untouched. It names no tools by design — in practice a stack prune touches zero lines of it, which is the check that the extraction into `references/` was done properly.

| Prune | Keep |
| --- | --- |
| `references/project-setup.md` — the *Elsewhere* gates column | `SKILL.md` — no tool names in it |
| `references/tool-defaults.md` — generic rows, replaced by pinned ones | `references/documentation.md` — arc42, Diátaxis and Mermaid are language-free |
| `references/code-and-tests.md` — other languages' formatters, test and mutation tools | `references/review.md` — the review brief is about conformance, not tooling |
| `references/release-and-automation.md` — the ecosystem table, down to your one release mechanism | `references/exploration.md` — brainstorming and spikes are stack-free |

**Read the rules you pin, because pinning exposes contradictions.** The test standards say migrations must never be verified against an embedded database — the example given is that a green migration test on H2 proves nothing about PostgreSQL. Pin a project whose *production* engine is that embedded database and the rule inverts: there, it is the real thing and the warning does not apply. A rule that was right in general can be wrong once the stack is fixed, and the prune is when you find out.

**Keep the copy derivable, not hand-maintained.** When the skill changes, go back to that same session — in the skill's repository — and ask for each project's copy to be re-derived rather than patched:

> The skill has changed. Update the copy at `<path-to-project>/.claude/skills/disciplined-delivery/` without re-adding anything we pruned there, and diff it afterwards to show me only the files that should differ.

Copying the stack-free files over wholesale and re-applying the prune to the rest is what keeps that cheap, and the diff is what proves nothing crept back in. Where a second project shares most of a stack, ask for its copy to be derived from the first rather than pruned again — that is what keeps two projects saying the same thing about the parts they share:

```
orders-service/.claude/skills/disciplined-delivery/    # Java · Maven · PostgreSQL · Angular
billing-portal/.claude/skills/disciplined-delivery/    # same, minus the frontend
```

The token saving is modest — most references shrink by well under a tenth. The point is different: a model that cannot see a Rust alternative cannot suggest one, and a reviewer reading the pinned copy sees the project's actual rules rather than a menu.

## Files

| File | Role |
| --- | --- |
| `SKILL.md` | The flow. Always loaded. Ten phases, the scope gate, hard stops. |
| `references/tool-defaults.md` | Which tool for which concern, and the bar each must clear. Read before the first tool decision of a session. |
| `references/project-setup.md` | Scaffolding, build quality gates, retrofit and init sequences, license compatibility. |
| `references/code-and-tests.md` | Code, comment, DDD, API-contract and test standards. Coverage ratchet. |
| `references/documentation.md` | arc42, Diátaxis, diagrams, screenshots, changelog, visual assets. |
| `references/review.md` | The review brief, handed to the review agent verbatim. |
| `references/release-and-automation.md` | Cutting a release, and turning repeated manual steps into portable scripts. |
| `references/exploration.md` | What a brainstorming session produces, and the per-branch permission a spike needs. |
| `docs/delivery-flow.md` | The flow as diagrams — Mermaid source, eleven of them. |
| `docs/diagrams/delivery-flow.md` | The same document with the fences replaced by rendered PNGs. **Open this one on GitHub**, which cannot render the ELK layout the source asks for. |

The reference files are read on demand, at the phase that needs them. Nothing in `docs/` is loaded by the skill — it is there for people, not for the model.

## Starting a new project

Open a session in the empty project folder. The first prompt can be short — say what you know and let the skill ask for the rest:

> Let's start a new project in this folder called **orders-service**: `<one line on what it does and for whom>`, based on `<Java 25 · Spring Boot · PostgreSQL · Angular>`. Use current stable versions — check them rather than relying on your training data. Base package `<com.example.orders>`. The remote repo should be `<owner/orders-service>`. Start step by step so we get fine-grained tickets.

That is enough to begin with. Everything else it asks for, and infers none of it:

1. **License** — required, and never chosen for you.
2. Build tool, target platform version, ticket system, CI system.
3. Which arc42 sections the project will use (full set for an application, a reduced set for a library or CLI).
4. Whether the project wants the **work-in-progress commit convention** — commit as soon as work exists, even untested, with `UNTESTED` / `WORKING` / `FIXED` on every subject; or plain Conventional Commits once a step is green. Off unless you ask for it.
5. **Accessibility and security requirements** — any target level, regulation, standard or customer obligation. Whatever you name becomes an NFR with measurable criteria, and only then is tooling chosen for it. "No special requirements" is recorded as an explicit answer rather than left silent.

Where it offers you a list to pick from, the list is never the whole answer: say something else in plain text, split an option, or ask a question back about the question. It asks again with what it learned rather than pushing your reply into one of its own boxes.

It then creates, in this order:

1. `README.md`, `LICENSE`, `CHANGELOG.md`, `.editorconfig`, `.gitattributes`, `docs/` skeleton.
2. The build file with quality gates wired in from the start — cheapest moment there is — plus hooks, and `commitlint` unless the work-in-progress commit convention was chosen.
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
7. `CHANGELOG.md` starting from the current version — history is not backfilled — and `commitlint` unless the work-in-progress commit convention is in use.
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

### You set how much it takes on

At the start of a session it asks one question — *proactive, or this task only?* — and carries the answer until you change it. **Task-only is what you get if it is never asked or never answered**: exactly the task you gave, followed by a list of what it would have done next, named rather than executed. Proactive lets it chain the obvious follow-on work without checking each time, though the scope gate, the hard stops and every confirmation the flow requires still apply to each item.

Neither setting changes how completely the task itself gets done — it bounds how many things are taken on, not how well the one you asked for is finished.

### It tells you when it is your turn

Every reply ends with **Action points** — a short list of what only you can do: review and merge a PR, answer a blocking question, make a decision. It comes last, after the summary and after any suggestions, and when a PR is waiting it is usually the single line. If nothing needs you, the block is absent rather than saying "none", so its presence is the signal.

### It pushes back before it complies

Agreement is not the default answer. When the approach, scope, tool or trade-off looks wrong, you get the concern, the alternative and what each costs — once, before the work happens, not as a hedge afterwards. Then it does what you decide, in full, and writes the accepted trade-off onto the ticket or into an ADR so the reasoning survives the conversation.

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

**These are defaults for a gap, not a migration target.** Whatever your project already uses overrules the table: an existing coverage tool, formatter or test framework stays, and replacing one is a work item with an ADR rather than something that happens on the side. The list decides what gets introduced where nothing exists yet.

| Area | Tool / standard | Why it is used | Role |
| --- | --- | --- | --- |
| Version control | **Git** | History as small coherent commits; merge commits, no squash | Default |
| Remote + tickets | **GitHub** (`gh`, GitHub Issues) | Remote repository, PRs, ticket system | Default — the accepted hosted exception |
| Ticket fallback | `docs/tickets/<id>.md` | Keeps work unblocked when no ticket system is reachable | Fallback only |
| Commit format | **Conventional Commits** | Machine-readable history, feeds changelog generation | Default |
| Commit gate | **commitlint** | Fails the build on a non-conforming commit message | Default — dropped only where the work-in-progress commit convention is chosen |
| Versioning | **SemVer** | Version numbers that mean something to consumers | Default |
| Changelog | **Keep a Changelog** | User-facing record of what changed per release | Default |
| Changelog drafting | **git-cliff** | Drafts entries from the commits; curated by hand afterwards | Default |
| Release automation | `semantic-release` and similar | Automated versioned releases | **Not a default** — requires squashed history; only on explicit request |
| Secret scanning | **gitleaks** (CLI or container) | Keeps credentials out of the diff and the history | Default — never the vendor's CI wrapper action, which is separately licensed |
| Local hooks | **pre-commit** | Fast local mirror of the build's gates | Default (CLI; the hosted service is not needed) |
| Build hygiene | **maven-enforcer-plugin** | Pins tool versions, converges dependencies, bans duplicates and snapshots | Default (Maven); Gradle version catalogs / locking, `npm ci`, `cargo deny` elsewhere |
| Coverage | **JaCoCo** with `check` bound to `verify` | Coverage measured *and* enforced, not just reported | Default (Maven); `cargo-llvm-cov`, `go test -coverprofile`, `coverage.py`, **Vitest** coverage (JS/TS), Coverlet elsewhere |
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
| JS / TypeScript tests | **Vitest** | Angular CLI default since v21; Karma and Jasmine are deprecated. Coverage built in, so no separate tool | Default (JS/TS) |
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
