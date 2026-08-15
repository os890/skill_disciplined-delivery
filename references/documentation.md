# Documentation, diagrams and visual assets

Read this at **phase 3** for the ADR and NFR sections, at **phase 7** for the changelog, and at **phase 9** for everything else. Check documentation for both **missing** and **outdated** content — a doc that describes last month's behaviour is worse than no doc.

## End-user documentation — `docs/user/`

Required for every user-visible change. A purely technical item may skip it; say so explicitly rather than skipping quietly.

Write for someone who does not know the codebase: what the feature is for, how to reach it, what they will see, what can go wrong and what to do about it. To the point, no filler — but do not omit a step because it seems obvious to you.

### Structure: Diátaxis

Four kinds of page, each serving a different need, **never mixed on one page** — the most common user-doc failure is a tutorial that turns into a reference halfway through:

| Kind | Answers | Shape |
| --- | --- | --- |
| **Tutorial** | "Get me started" | A lesson that must work start to finish, one path, no options |
| **How-to guide** | "How do I do X?" | Task-focused steps for someone who already has a goal |
| **Reference** | "What exactly does this do?" | Complete, dry, structured; generated where possible |
| **Explanation** | "Why is it like this?" | Background and reasoning; links the ADR |

When a page starts serving two needs, split it. `docs/user/` mirrors these four as its top-level structure.

It must be visual:

- **Shipped UI → screenshots**, captured while writing the docs against the actual build. Not from memory, not reused from an older version.
- Prefer a **scripted capture** — a Playwright script under `docs/screenshots/`, committed with the docs — so every screenshot can be regenerated when the UI moves instead of rotting in place. Run it in a container if Playwright is not installed.
- **Draft, or UI that does not exist yet → a UI mockup**, marked as a draft in the surrounding text so no reader mistakes it for the shipped product. Replace it with real screenshots once the UI exists.
- Annotate what matters (the button, the field, the error) and crop to the region under discussion — no full-desktop captures.
- **Never** put a customer name, real personal data, token, licence key or internal hostname in a screenshot. Use seeded demo data.

## Technical documentation — `docs/technical/`

Always, and always with diagrams. Cover what a newcomer cannot get from the code quickly: entry points, the flow, the invariants, the failure modes, how to run and test it, and *why* it is shaped this way — linking the ADR rather than paraphrasing it.

### Structure: arc42

**arc42 is the default structure** (Markdown variant, one file per section under `docs/technical/arc42/`). It is a system-level document that work items *update*; never create a new arc42 skeleton per ticket.

| # | Section | What goes in it | Default diagram |
| --- | --- | --- | --- |
| 1 | Introduction and Goals | Purpose, top quality goals, stakeholders | — |
| 2 | Architecture Constraints | Technical and organisational constraints, mandated platforms | — |
| 3 | Context and Scope | Neighbouring systems, users, external interfaces | `flowchart` in C4-context style |
| 4 | Solution Strategy | The few decisions that shape everything else, each linking an ADR | — |
| 5 | Building Block View | Decomposition, level by level; bounded contexts | `flowchart` / `architecture-beta` per level |
| 6 | Runtime View | The important scenarios, including failure paths | `sequenceDiagram`, `stateDiagram-v2` for lifecycles |
| 7 | Deployment View | Environments, containers, topology | `flowchart` with subgraphs per node |
| 8 | Cross-cutting Concepts | Domain model, persistence, security, error handling, logging, i18n | `classDiagram` for the model, `erDiagram` for data |
| 9 | Architecture Decisions | **Index linking `docs/adr/NNNN-*.md`** — never a second copy of the decisions | — |
| 10 | Quality Requirements | Quality tree plus scenarios; **links `docs/requirements/nfr.md`** | — |
| 11 | Risks and Technical Debt | What phase 10 found and the user accepted, with the ticket for each | — |
| 12 | Glossary | The ubiquitous language: domain terms with their exact meaning | — |

Rules that keep it useful:

- **Filled or absent — never present-and-empty.** A stub full of "TBD" reads as documented and isn't. If a section is expected but does not apply, write one line: "not applicable, because …".
- **No duplication.** §9 and §10 link the ADR files and the NFR file. ADRs stay separate, numbered and immutable so they can be referenced from code and commits.
- **Grow on demand.** Start with the sections the project actually has answers for; add a section when a work item produces content for it.
- **Small projects get a reduced set.** For a library or CLI, §1, §3, §5, §8, §9, §12 are usually enough — or just `README.md` plus ADRs. Record which set the project uses in an ADR.
- arc42 is free to use with attribution — keep the template's attribution note, and check the current terms on `arc42.org` rather than assuming them.

**arc42 is the architecture document, not all technical documentation.** Anything it has no home for lives directly in `docs/technical/` beside it, linked from the `README.md` and from the arc42 section it belongs to:

| Document | Purpose |
| --- | --- |
| `docs/technical/runbook.md` | Operating the system: start, stop, health checks, alerts, known failure modes and their remedies, rollback. |
| `docs/technical/api.md` | API / interface reference, or a pointer to the generated one. |
| `docs/technical/development.md` | Getting a developer productive: prerequisites, build, run, test, debug, how to run the container-based tests. |

Generated content (OpenAPI, Javadoc, coverage reports) is linked, never copied by hand into Markdown — a hand-copied API reference is wrong within a week.

### Diagrams

**Mermaid is the default.** Use PlantUML only where the project already standardises on it. Beyond the per-section defaults above: `flowchart` for branching logic, `block-beta` for a UI or layout sketch, `gantt` or `gitGraph` for a delivery timeline.

Keep the diagram source inline in the Markdown so it diffs and reviews like code. One diagram that shows the real mechanism beats three decorative ones — if a diagram only restates the section heading, delete it.

Add a one-line caption saying what the reader should take from the diagram; it doubles as the accessible description.

**Look at the rendered diagram before committing it** — parsing is not the same as reading, and layout defects only show up in the image. Render without installing anything, per the container rule in `SKILL.md`; given a Markdown file this writes every fence to `doc-N.png` plus a copy with image links:

```sh
podman run --rm --userns=keep-id -v "$PWD:/data:z" docker.io/minlag/mermaid-cli \
  -i doc.md -o rendered/doc.md -e png -s 3 -b white
```

*Hint for a tangled diagram:* overlapping edges are usually the layout engine, not the diagram. Mermaid's default (dagre) routes splines and ignores crossings; adding `config: {layout: elk, elk: {mergeEdges: true}}` to the fence's frontmatter routes orthogonally and merges edge fans instead. Two catches: ELK places the entry node mid-canvas if the flow has back-edges (make it acyclic), and **GitHub does not register the ELK plugin** — it silently falls back to dagre, so commit the ELK-rendered PNG for any diagram whose readability depends on it.

## Visual assets in Git

Prefer formats the platform renders straight from the repository, with no build step and no external service, so the docs work inside the PR review itself:

| Need | Format | Editable with |
| --- | --- | --- |
| Any diagram | ```` ```mermaid ```` fence | any editor; rendered by GitHub |
| Rough UI layout as code | ```` ```mermaid ```` + `block-beta` | any editor |
| Draft wireframe / mockup | `*.excalidraw.svg` (scene embedded in the SVG) | Excalidraw (MIT, self-hostable), or its VS Code extension |
| Shape-library mockup | `*.drawio.svg` (source embedded in the SVG) | draw.io desktop (Apache-2.0), or its VS Code extension |
| High-fidelity design | SVG/PNG exported from Penpot (MPL-2.0, self-hostable) | Penpot |
| Throwaway sketch in a PR comment | ASCII art in a plain code fence | any editor |

Rules for committed images:

- Reference images by **relative repository path**; externally hosted SVG is often blocked.
- Keep them **editable**: the `.excalidraw.svg` / `.drawio.svg` dual formats render as images while staying re-openable in their editor, so a mockup never becomes a dead artifact.
- For light/dark support, commit both variants and select with `<picture>` + `prefers-color-scheme`.
- Every image gets alt text that carries the information, not the filename.
- `block-beta` is a newer Mermaid diagram type and GitHub lags Mermaid releases — verify it renders with one throwaway commit before a project relies on it. Embedded fonts in SVG may be dropped by the platform's sanitizer; export text as paths where fidelity matters.
- Avoid PlantUML in Markdown unless the project renders it in CI and commits the output; GitHub does not render it.

## Architecture decision records

`docs/adr/NNNN-<title>.md`, one decision per file, using the **MADR** template: title, status, context and problem, considered options, decision outcome with its justification, consequences (good and bad).

- **Immutable.** A decision that no longer holds is marked `superseded by ADR-00NN`; it is never rewritten, because the reasoning at the time is the value.
- One decision per ADR. "Our stack" is not a decision, it is a list.
- Record the options you rejected and why — an ADR without rejected options is an announcement, not a decision record.
- Referenced from the code and the commits by number, and indexed from arc42 §9.

### What earns an ADR

**Not every decision does**, and the set has a cost: every session reads the ADR *titles* while orienting, so each record that is not architectural makes the ones that are harder to find. Fifty files about tool trivia bury the three that shape the system.

Write one when the decision does at least one of these:

- **Constrains future work** — other code has to be written differently because of it.
- **Is expensive to reverse** — a migration, a rewrite, or a coordinated change across modules.
- **Would leave a newcomer asking "why is it like this?"** with no answer visible in the code.

The test: *what else would have to change if this were reversed next month?* Nothing outside the file it lives in — it is not an ADR.

Decisions that usually fail that test: the build wrapper (`mvnw`), a plugin version, a formatter setting, a directory name, a library pulled in for one utility, a naming convention the linter already enforces. They are still decisions and rule 1 still applies, so they are written down — just where they are used: the build file, `docs/technical/development.md`, the ticket, or a comment at the site.

**And it has to be a decision, not a description.** This is the more common mis-filing: the subject really is architectural, so it looks like ADR material, but what gets written is how the system *is* rather than a choice that was made. An ADR is frozen the day it is written; arc42 is kept current. So the question is what happens to the text later — if the system changed next year, an ADR would still be true, because it records a choice made at a moment. A description would simply be wrong, which is why it belongs where it can be maintained.

| What you are writing | Where it goes |
| --- | --- |
| "We chose X over Y and Z, because …, and it costs us …" | **ADR** |
| "The system is structured as …", "errors are handled by …", "the domain model is …" | **arc42** §5 building blocks, §7 deployment, §8 cross-cutting concepts |
| "This uses the settlement date, not the booking date" | a **comment** at the site |
| "p95 under 200 ms" | an **NFR** |

Two smells that it is not an ADR: the draft has **no rejected options** — a description wearing an ADR's shape — and you would have to **edit it when the code changes**, which an immutable file cannot support.

Where a decision genuinely shaped the structure, write both: the ADR for the choice, the arc42 section for the result. §9 links them and neither repeats the other.

**Calibrate against what the system is for.** Neither test above is absolute, because the same subject is core in one product and plumbing in another. Build tooling is an architecture decision for a build plugin and plumbing for a business app. Transaction boundaries are central where the domain has invariants to protect and incidental in a read-only viewer. A retry-and-backoff policy is the heart of an integration gateway and noise in an internal CRUD screen. Copying another project's ADR set is how a domain-heavy application ends up with records about its wrapper script and none about its aggregates.

The yardstick is already written down: **arc42 §1** — the system's purpose and its top quality goals — which every session reads while orienting anyway. If a decision touches neither what the system exists to do nor a quality goal it is judged on, it is probably plumbing, and it goes where it is used.

Borderline, prefer the ticket. An ADR can always be written later once a decision proves to shape something, whereas an ADR is immutable — a premature one cannot be deleted, only superseded, and the noise is permanent.

## Changelog

`CHANGELOG.md` in **Keep a Changelog** form, with **SemVer** version headings and an *Unreleased* section that is always current.

- Draft it from the Conventional Commits (`git-cliff`), then **curate it for humans** — a generated file pasted unedited is a commit log with extra steps.
- Written from the user's point of view: what changed for them, not which class was touched.
- Breaking changes are called out with a migration note. That note is the thing users actually need and the thing most often missing.
- Grouped as Added / Changed / Deprecated / Removed / Fixed / Security.

## Keeping docs honest

- Documentation changes ride **in the same PR** as the code they describe. A follow-up ticket for docs is how docs go stale.
- When a change makes existing documentation wrong, fixing it is part of the work item, not a nice-to-have.
- If you find outdated content outside the scope of the current item, report it and offer a ticket — do not silently expand the change.
- **markdownlint** and **lychee** run as build gates: dead links and broken structure are the cheapest rot to catch and the most common. A link to a moved page erodes trust in the whole document.
- Generated documentation (OpenAPI, Javadoc, coverage) is regenerated by the build and linked — never hand-copied into Markdown.
