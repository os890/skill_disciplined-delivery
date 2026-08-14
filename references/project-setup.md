# Project setup and build quality gates

Read this when initializing a project or when a phase finds the scaffolding or gates missing — and also, at any point in a work item, for **§ License compatibility** and **§ Tools that do not clear the bar** when a tool or dependency is being proposed or a scan reports a finding, for **§ Rules for the gates** when a gate fails, and for **§ Dependency updates** when the item is a dependency bump.

**When something here is absent** — no ADR folder, no coverage gate, no formatter config, no license — report what is missing and propose creating it. Do not invent structure silently, and do not drop the rule because the structure is absent.

Which route depends on what is missing: a missing **file or folder** (docs skeleton, `CHANGELOG.md`, `.editorconfig`) can ride along with the current work item. A missing **gate** gets its own ticket and its own commit, because a gate plus a behaviour change in one diff is unreviewable — see *Retrofitting an existing project* below.

## Scaffolding

Every project carries these at the **repository root** — and in a repository holding several projects, at **each project's root**:

| File / folder | Rule |
| --- | --- |
| `README.md` | Required. What it is, how to build, how to run, how to test, where the docs live, how to contribute. |
| `LICENSE` | Required. **The user chooses the license when the project starts — never pick one for them.** |
| `CHANGELOG.md` | Required. Keep a Changelog format, SemVer headings, an *Unreleased* section that is always current. |
| `.editorconfig` | Required. Indentation, charset, final newline — stops IDE-driven whitespace churn. |
| `.gitattributes` | Required. `* text=eol=lf` plus binary markers, so line endings never depend on the developer's OS. |
| `.gitignore` | Required. Build output, IDE files, local configuration — so a `git status` is meaningful and no artifact is committed by accident. |
| `.pre-commit-config.yaml` (or hooks) | Fast local mirror of the build's gates: formatter, `gitleaks`, and `commitlint` where it is in use. |
| Lockfile | Committed, so a build is reproducible and an update is a visible diff. |
| `docs/user/` | End-user documentation. |
| `docs/technical/` | Technical documentation, with diagrams. |
| `docs/technical/arc42/` | The arc42 sections (default structure — see `documentation.md`), one file per section. |
| `docs/technical/runbook.md`, `api.md`, `development.md` | Technical docs arc42 has no home for: operations, interface reference, developer setup. |
| `docs/adr/NNNN-<title>.md` | Architecture decision records, sequentially numbered, never rewritten — superseded by a newer ADR instead. |
| `docs/requirements/nfr.md` | Non-functional requirements, each with a measurable number. |
| `docs/plans/<ticket>.md` | The step-by-step plan per work item. |
| `docs/screenshots/` | The screenshot capture script and its output. |
| `scripts/` | Automation for anything no tool covers — see `release-and-automation.md`. |
| `docs/tickets/<id>.md` | Only as the fallback when no ticket system is reachable. |

### License enforcement

A license rule nobody enforces decays. Guard it in the build:

- **Maven** — `apache-rat-plugin`, bound to `verify`, failing on missing or wrong file headers.
- Elsewhere — `reuse lint` (REUSE spec), `cargo deny check licenses`, `license-checker`, or the build tool's equivalent.

The same applies to *incoming* licenses: dependency licenses must be checked against what the project's own license permits.

### License compatibility

Only one distinction decides compatibility:

| Category | Does its license affect your artifact? |
| --- | --- |
| Build, test and CI tooling — Maven plugins, linters, scanners, Playwright, Podman, Testcontainers, doc tools | **No.** You run it; you do not ship it. Its license governs the tool, not your output. |
| Shipped runtime dependencies | **Yes.** This is the only category where compatibility is actually decided. |
| Code or prose you copy in — a snippet, a template's text | **Yes**, as a derivative work. |

**For an Apache-2.0 project specifically:**

- Ship a `NOTICE` file and a license header in every source file; the RAT gate is what keeps that true.
- Runtime dependencies under Apache-2.0, MIT, BSD, ISC or similar permissive terms are unproblematic. MPL-2.0 and EPL-2.0 are usable as unmodified binary dependencies but do not permit copying their code into your files.
- **GPL, AGPL and LGPL runtime dependencies are the real hazard** — an Apache-2.0 artifact cannot absorb GPL-licensed code. Treat any such finding as a blocker: report it and let the user decide, never resolve it alone.
- Copyleft **build-time** tooling is fine. Several defaults are copyleft-licensed — JaCoCo (EPL-2.0), SpotBugs and Checkstyle (LGPL-2.1), golangci-lint (GPL-3.0), the FSFE `reuse` tool (GPL-3.0-or-later) — and none of them is part of the released artifact, so none affects its license. Do not vendor their source into your own, and do not redistribute a modified build of them without following their terms.
- **Documentation templates.** arc42 and Diátaxis are CC BY-SA 4.0; MADR is MIT or CC0. The share-alike term binds redistributing or adapting *the template itself*, not the content you write into it — arc42's own FAQ is explicit that a filled-in architecture document may stay proprietary and confidential. So: keep the attribution the template asks for, and strip the template's explanatory guidance text out of the project's copy rather than shipping it as if it were yours. Your own words are yours.
- Verify each tool's and template's **current** license before adoption. Licenses change, and this is exactly the check the project's own license gate cannot do for you.

### Tools that do not clear the bar

Recorded so they are not reintroduced by habit. Any of them needs an explicit decision by the user, not a default:

| Tool | Why not |
| --- | --- |
| **Docker Desktop** | Proprietary EULA, and a paid subscription is required above a company-size or revenue threshold. Podman, or Docker Engine/CLI, does the same job under Apache-2.0. |
| **Flyway Teams/Enterprise** | Per-user licence. Community is the default; design releases so no paid feature is needed. |
| Hosted siblings of free tools — WireMock Cloud, PactFlow, Testcontainers Cloud, Excalidraw+, `pre-commit.ci`, vendor CI wrapper actions | Each has a fully capable self-hosted or CLI counterpart. Reach for the free one; the hosted version buys collaboration features, not capability. |

## Build quality gates

The build enforces the rules of this skill — a rule only a human remembers is a rule that decays. Wire the quality plugins the project's build tool offers, unless an **ADR** records a different decision.

| Concern | Maven default | Elsewhere |
| --- | --- | --- |
| Build and dependency hygiene | **maven-enforcer-plugin** — `requireMavenVersion`, `requireJavaVersion`, `dependencyConvergence`, banned and duplicate dependencies, no snapshot deps in releases | Gradle dependency locking / version catalogs, `npm ci` with a committed lockfile, `cargo deny check` |
| Coverage with a failing threshold | **jacoco-maven-plugin**, `check` rules bound to `verify` — threshold 80 % floor, ~90 % on new code, ratcheted where the repo starts below (see `code-and-tests.md` § Coverage) | `cargo-llvm-cov`, `go test -coverprofile` + threshold check, `coverage.py --fail-under`, **Vitest** coverage (v8) for JS/TS, Coverlet |
| Formatting and linting | **Spotless** (`spotless:check`), Checkstyle where the project uses it | `ruff`, `gofmt -l` + `golangci-lint`, `clippy -D warnings`, ESLint + Prettier, `dotnet format --verify-no-changes` |
| Static analysis / bug patterns | **SpotBugs**, Error Prone (+ NullAway) | `golangci-lint`, `clippy`, `mypy`/`pyright`, `tsc --noEmit`, Roslyn analyzers |
| Architecture and DDD boundaries | **ArchUnit** tests | `import-linter` (Python), `eslint-plugin-boundaries`, `deptrac` |
| Dependency vulnerabilities | **dependency-check**, or the platform's advisory scan | `pip-audit`, `cargo audit`, `npm audit`, Trivy / Grype |
| License compliance | **apache-rat-plugin** | `reuse lint`, `cargo deny check licenses`, `license-checker` (use a maintained fork — the original is stale) |
| Secrets in the diff and history | **gitleaks** — the CLI or its container image, **not** the vendor's GitHub Action wrapper, which is licensed separately and is free only for a single repository | same tool everywhere |
| Commit message conformance | **commitlint** in a hook and in CI — unless the work-in-progress convention is on, which drops it | same tool everywhere |
| SBOM generation | **cyclonedx-maven-plugin** | `cyclonedx-*` for the ecosystem, `syft` |
| API contract linting | **Spectral** on the OpenAPI / AsyncAPI file, `oasdiff` for breaking changes | same tools everywhere |
| Documentation rot | **markdownlint**, **lychee** for dead links | same tools everywhere |
| Public API compatibility (libraries) | **japicmp** or **revapi** | `cargo-semver-checks`, `api-extractor` |
| Shell script correctness | **shellcheck** on everything in `scripts/` | same tool everywhere |
| Mutation testing (optional) | **pitest** | Stryker, `cargo-mutants` |

### Rules for the gates

- **Every gate fails the build.** A warning nobody reads is not a gate. Reports are welcome as extra output, never as the only outcome.
- **The same gates run locally and in CI** — one `mvn verify` (or its equivalent) reproduces what CI does. No CI-only check a developer cannot run.
- **Pin plugin and tool versions** so the build is reproducible and a gate cannot change under you.
- **Run the upstream CLI or container image, never a CI-platform wrapper.** Wrapper actions and plugins are separately licensed from the tool they wrap and have gone commercial while the tool itself stayed free. Using the CLI also keeps the gate identical locally and in CI, and portable if the CI platform changes.
- **Adding, weakening or removing a gate is an architecture decision** → it needs an ADR, not a quiet edit to the build file. That includes lowering a threshold to make a build pass.
- Keep the build fast enough to actually be run: slow suites belong behind a profile or a separate stage — but **still inside the gate**. A stage that a PR can merge without is not a gate, it is a report. Phase 6 is not green until every stage is.

### Local hooks are a mirror, not the authority

Hooks (`pre-commit`, `gitleaks`, and `commitlint` where it is in use) exist to fail in two seconds instead of two minutes. Everything they check is **also** checked by the build, because a hook can be skipped with `--no-verify` and a fresh clone may not have them installed. Never move a check into a hook only.

Run the hook framework locally as a CLI. Its hosted counterpart is a separate product that is not free for private repositories, and it is not needed: the build already runs the same checks.

## Dependency updates

Dependencies rot whether or not anyone is watching, so the update is a scheduled activity rather than a reaction to an incident. No bot is required — the test suite is what makes an update safe, and the vulnerability scanner is what makes it urgent.

- **A recurring ticket** (monthly, or per sprint) to review and lift dependencies, grouped by ecosystem so each PR stays reviewable.
- A vulnerability finding in the scanner gate is **not** a recurring ticket — it is its own ticket, prioritised by severity and exploitability in this project's actual usage.
- Where the project's platform already offers update PRs, use them rather than adding tooling; the same rules apply to the PRs they open.
- Patch and minor updates of **dev-time** dependencies are the archetypal **reduced-path** work item: no behaviour change, existing tests cover it. They still need a green build and a real look at what changed.
- **Runtime dependencies and every major version get a human**, no matter how green the build is.
- A major update that requires code changes is not a dependency bump: it gets its own ticket and the full flow.
- Never lift a version just to quiet a scanner without reading what actually changed.
- Keep the lockfile committed, so an update is a reviewable diff rather than an invisible drift.

## Supply chain

- **SBOM** (CycloneDX) generated by the build and attached to every release, so "what is in this artifact" is answerable after the fact.
- Dependency licenses checked against the project's own license, not just scanned for vulnerabilities.
- Where the project ships to third parties or into the EU market, treat SBOM, vulnerability handling and update policy as product requirements with deadlines rather than hygiene — verify the current obligations for that market instead of assuming.

## Retrofitting an existing project

Introduce gates **one at a time, each on its own ticket**, so every diff stays reviewable and a failure has one obvious cause. A useful order:

1. `.editorconfig`, `.gitattributes`, then the formatter (largest diff, zero behaviour change — get it out of the way first).
2. `gitleaks` over the working tree *and the history* — findings here are urgent and rewrite-or-rotate decisions, not backlog items.
3. Enforcer / dependency hygiene.
4. Coverage measurement, gate set to the current number (see the ratchet in `code-and-tests.md`).
5. Static analysis, with a baseline for existing findings and zero tolerance for new ones.
6. Architecture tests, encoding the boundaries the code already respects.
7. `CHANGELOG.md`, starting from the current version — do not backfill history — plus `commitlint`, unless the project uses the work-in-progress commit convention.
8. Dependency and license scanning, SBOM, then the recurring dependency-update ticket once the suite is trustworthy enough to make an update safe.

Never introduce a gate and a behaviour change in the same commit.

## Initializing a new project

Order that avoids rework:

1. Ask the user: license, build tool, target platform version, ticket system, CI system, which arc42 sections the project will use, the **accessibility and security requirements** (below), and whether the project wants the **work-in-progress commit convention** (below). **Do not choose the license.**
2. `README.md`, `LICENSE`, `CHANGELOG.md`, `.editorconfig`, `.gitattributes`, `docs/` skeleton.
3. Build file with the quality gates wired from the start — cheapest moment there is — plus hooks, the release mechanism from `release-and-automation.md`, and `commitlint` unless answer 1 chose the work-in-progress commit convention.
4. `ADR-0001` recording the stack, the gate set, which arc42 sections the project uses and whether the work-in-progress commit convention is on, so later deviations have something to deviate from.
5. `docs/requirements/nfr.md` with whatever numbers are known, even if sparse, and the arc42 sections you can already fill — §1, §2, §3 and §12 usually exist on day one. Leave the rest out until there is content.
6. CI running the same build the developer runs.
7. A walking skeleton — one thin end-to-end path with a test — before any feature work, so the test infrastructure is proven rather than assumed.

### The work-in-progress commit convention

Not every team wants it, so **ask at project start and record the answer in `ADR-0001`** rather than assuming either way:

> Commit as soon as work exists — even untested — with a status token on every subject (`UNTESTED` / `WORKING` / `FIXED`), promoted before pushing? Or commit only once a step is green, with plain Conventional Commits?

- **On** — the rules in `SKILL.md` phases 5 and 7 apply, and **`commitlint` is dropped**: a leading status token is exactly what its parser rejects, and reconfiguring it buys less than it costs. The trade is real and worth naming — nothing then enforces the commit format, and `git-cliff` cannot draft the changelog from subjects it cannot parse, so the `CHANGELOG.md` entry is written by hand. Both are acceptable; neither should be a surprise.
- **Off** — the ordinary model, and the one everything else in this skill is built around: commit when the step is green, plain Conventional Commits enforced by `commitlint`, `git-cliff` drafting the changelog, and nothing in review looking for a token.

**Off is the default for a project that never answered**, including an existing one adopting this skill. It is an offer, not something to impose on a history that already reads differently.

### Accessibility and security requirements

Neither is assumed and neither is imposed. **Ask at project start whether the project has special requirements**, because both are cheap to design in and expensive to retrofit:

- **Accessibility** — is there a target level or regulation to meet (a WCAG conformance level, a public-sector or EU market obligation, an internal standard), and for which surfaces?
- **Security** — is there a standard, regulation or customer obligation in play (a requirements catalogue, a threat-modelling expectation, audit or certification needs, data-classification rules)?

Then:

- Requirements the user names become **NFRs in `docs/requirements/nfr.md` with measurable criteria**, and the tooling to verify them is chosen at that point — not before. A tool without a requirement behind it is scope creep.
- If the answer is "no special requirements", **record that answer** in the NFR file. An explicit "none stated for this project" is a decision; silence is an assumption, and rule 2 forbids assumptions.
- Either way, phase 2 keeps asking per work item — a single item can carry a security or accessibility implication the project-level answer never anticipated.
