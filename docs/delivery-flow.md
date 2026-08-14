# Disciplined Delivery — the approach, visualized

Seven views of the same discipline, nine diagrams. Source is inline Mermaid so it diffs and reviews like code; the rendered PNGs live in `docs/diagrams/`, regenerated from the skill root with:

```sh
podman run --rm --userns=keep-id -v "$PWD:/data:z" docker.io/minlag/mermaid-cli \
  -i docs/delivery-flow.md -o docs/diagrams/delivery-flow.md -e png -s 3 -b white
```

The flowcharts declare `layout: elk` in their frontmatter: ELK routes edges orthogonally and minimises crossings, which the default (dagre) engine does not.

**GitHub does not register the ELK plugin**, so the fences below fall back to dagre there and render with crossings — nothing errors, it just looks worse. [`docs/diagrams/delivery-flow.md`](diagrams/delivery-flow.md) is this same document with the fences replaced by the ELK-rendered images, and is the one to open on GitHub.

---

## 1. End to end: from a topic to a routed piece of work

*Take away: nothing reaches a phase number before the understanding is confirmed and the path is agreed.*

```mermaid
---
config:
  layout: elk
  elk:
    mergeEdges: true
    nodePlacementStrategy: BRANDES_KOEPF
---
flowchart TD
    A([New topic arrives]) --> B["<b>Orient</b><br/>CLAUDE.md · ADR titles · NFRs<br/>arc42 §1/§3/§5 · build gates and thresholds"]
    B --> C[/"<b>Mirror the topic back</b><br/>goal · who for · assumed state<br/>in / out of scope · risks<br/><i>then WAIT</i>"/]
    C -->|"picture is wrong"| C
    C -->|confirmed| D{Kind of request?}

    D -->|Brainstorming| E["No files changed · no branch<br/>produces: options and trade-offs,<br/>open questions, candidate ADRs / NFRs"]
    E --> F

    D -->|"PoC / spike"| G{"Explicit permission<br/>for <i>this one</i> branch?"}
    G -->|no| HS(["HARD STOP — ask first"])
    G -->|yes| H["branch <code>spike/topic</code><br/>flow suspended · time-boxed<br/>code is throwaway, knowledge is the deliverable"]
    H --> I["Write down what was learned<br/>PR marked <i>not for merge</i>"]
    I --> F

    D -->|Work item| F["<b>Candidate work item</b>"]

    F --> SG{"<b>Scope gate</b><br/>proposed, never silent"}
    SG -->|"no visible behaviour change · no API change<br/>no schema · no new dependency<br/>existing tests cover it"| RED["<b>Reduced path</b><br/>skips 3, 4, 9, 10<br/>+ conformance half of 8"]
    SG -->|"bug: correct behaviour<br/>already defined somewhere"| BUG["Reproduce with a failing test<br/>fix · changelog entry"]
    SG -->|"bug: correct behaviour<br/><b>not</b> defined anywhere"| FULL
    SG -->|anything else| FULL["<b>Full flow</b> — the default"]

    RED --> CONF[/"Name the skipped phases · WAIT for confirmation<br/>record the decision on the ticket"/]
    BUG --> CONF
    CONF --> PH([Enter the phases])
    FULL --> PH

    style HS fill:#c0392b,color:#fff
    style C fill:#2c3e50,color:#fff
    style SG fill:#8e44ad,color:#fff
```

---

## 2. The ten phases and what each one leaves behind

*Take away: every phase produces a durable artifact — the phase is not "done thinking", it is "done writing".*

```mermaid
---
config:
  layout: elk
  elk:
    mergeEdges: true
    nodePlacementStrategy: BRANDES_KOEPF
---
flowchart TD
    P1["<b>1 · Definition + ticket</b>"] --> P2["<b>2 · Clarification loop</b><br/>scope · edges · data · NFRs<br/>integration · operations"]
    P2 --> P3["<b>3 · Anchor requirements</b>"]
    P3 --> P4["<b>4 · Plan</b>"]
    P4 --> P5["<b>5 · TDD implementation</b><br/>red → green → refactor"]
    P5 --> P6["<b>6 · Build gate</b>"]
    P6 --> P7["<b>7 · Commit + PR</b>"]
    P7 --> P8["<b>8 · Review</b>"]
    P8 --> P9["<b>9 · Documentation</b>"]
    P9 --> P10["<b>10 · Production readiness</b>"]
    P10 --> DONE([Done])

    P1 -.-> A1["GitHub Issue<br/><i>or</i> docs/tickets/id.md<br/>+ ticket-named branch"]
    P2 -.-> A2["Refined ticket<br/><i>no material question open</i>"]
    P3 -.-> A3["Given/When/Then criteria → ticket<br/>structural choice → MADR ADR<br/>quality attribute → NFR <b>with a number</b><br/>consumed interface → OpenAPI / AsyncAPI"]
    P4 -.-> A4["docs/plans/ticket.md<br/>committed <b>before</b> code<br/>+ verification answered aloud"]
    P5 -.-> A5["Small coherent commits<br/>suite green at each one"]
    P6 -.-> A6["Actual build output,<br/>failures included"]
    P7 -.-> A7["Conventional Commits + ticket no.<br/>CHANGELOG entry<br/>merge commits, no squash"]
    P8 -.-> A8["Mechanical pass + <b>conformance</b> agent<br/>findings reported, never silently fixed<br/>each triaged: fix / ticket / reject"]
    P9 -.-> A9["Diátaxis user docs + real screenshots<br/>arc42 sections + Mermaid<br/>checked for <i>outdated</i>, not just missing"]
    P10 -.-> A10["Accepted risks → arc42 §11<br/>each with a ticket"]

    classDef art fill:#ecf0f1,stroke:#95a5a6,color:#2c3e50;
    class A1,A2,A3,A4,A5,A6,A7,A8,A9,A10 art
```

---

## 3. The core invariant: no invisible decisions

*Take away: this loop runs continuously underneath every phase — it is the skill, the phases are just where it shows up.*

```mermaid
---
config:
  layout: elk
  elk:
    mergeEdges: true
    nodePlacementStrategy: BRANDES_KOEPF
---
flowchart LR
    D(["A decision is<br/>about to be made<br/><i>default · limit · timeout ·<br/>rounding · retry count</i>"]) --> Q{"Backed by a<br/>written definition?"}

    Q -->|"acceptance criterion"| GO
    Q -->|ADR| GO
    Q -->|NFR| GO
    Q -->|"explicit user answer"| GO(["Implement it"])

    Q -->|"no — or only a<br/><i>reasonable default</i>"| S["<b>STOP</b><br/>state the blocker,<br/>batch the question"]
    S --> K["Keep working on everything<br/>independent of the answer"]
    S --> W["Answer → written into<br/>ticket / ADR / NFR"]
    W --> GO
    K -.-> W

    style S fill:#c0392b,color:#fff
    style GO fill:#27ae60,color:#fff
```

Hard stops that trigger the same loop:

```mermaid
---
config:
  layout: elk
  elk:
    mergeEdges: true
    nodePlacementStrategy: BRANDES_KOEPF
---
flowchart LR
    HS{{"<b>Stop and<br/>involve the user</b>"}}
    HS --- H1["A phase-2 question is still open"]
    HS --- H2["A decision has no backing definition"]
    HS --- H3["An <b>existing test</b> would have to change"]
    HS --- H4["Plan and definitions disagree"]
    HS --- H5["Suite red, or a quality gate fails"]
    HS --- H6["A phase would be skipped unconfirmed"]
    HS --- H7["A license must be chosen"]
    HS --- H8["A third-party dependency would be added"]
    HS --- H9["Spike starts without permission,<br/>or spike code reaches mainline"]
    HS --- H10["Brainstorming becomes implementation"]
    HS --- H11["Shared understanding not yet confirmed"]

    HS --> MEAN["<b>Stopping means:</b> state the blocker and what you need ·<br/>continue everything independent of it ·<br/><b>never</b> substitute a guess"]

    style HS fill:#c0392b,color:#fff
    style MEAN fill:#ecf0f1,stroke:#95a5a6,color:#2c3e50
```

---

## 4. Where state lives, and how a session resumes

*Take away: the live task list is a view, never the record — a resumption is reconstructed from the repository and verified by running the build.*

```mermaid
---
config:
  layout: elk
  elk:
    mergeEdges: true
    nodePlacementStrategy: BRANDES_KOEPF
---
flowchart TB
    subgraph LIVE["Live — dies with the session"]
        TL["Harness task list<br/><i>one entry per applicable phase</i>"]
    end

    subgraph DUR["Durable — survives the session"]
        TK["<b>Ticket</b><br/>definition · criteria<br/>confirmed understanding<br/>scope-gate decision · blockers"]
        PL["<b>docs/plans/ticket.md</b><br/>step checkboxes = progress record"]
        AD["<b>docs/adr/</b> — decisions, immutable"]
        NF["<b>docs/requirements/nfr.md</b>"]
        GT["<b>Branch · commits · PR</b>"]
    end

    TL -.->|"anything that must outlive<br/>the session is written here"| DUR

    DUR ==> R1
    subgraph RESUME["Reconstructing a topic, in order"]
        R1["1 · the <b>branch</b> names the ticket"] --> R2["2 · the <b>ticket</b>"]
        R2 --> R3["3 · the <b>plan file</b> checkboxes"]
        R3 --> R4["4 · <b>git log</b> and PR state"]
        R4 --> R5["5 · <b>run the build</b><br/><i>what the suite reports beats<br/>what the plan claims</i>"]
    end
    R5 --> REP[/"Report the reconstructed state<br/>and WAIT before resuming"/]

    style LIVE fill:#fdf2e9,stroke:#e67e22
    style DUR fill:#eafaf1,stroke:#27ae60
    style R5 fill:#8e44ad,color:#fff
```

---

## 5. Quality is the gate, not the goal

*Take away: red is always fixed in the code — the two tempting shortcuts both require the user's agreement first.*

```mermaid
---
config:
  layout: elk
  elk:
    mergeEdges: true
    nodePlacementStrategy: BRANDES_KOEPF
---
flowchart TD
    B(["Run the full build<br/><i>compile · tests · every quality gate</i>"]) --> R{Result?}
    R -->|"all pass"| G(["<b>Green</b> — phase 6 complete"])
    R -->|"a gate or a test fails"| RED["<b>Red</b>"]

    RED --> ROUTE{"Which route?"}

    ROUTE -->|"<b>the only default</b>"| F["<b>Fix the code</b>"]
    ROUTE -->|"weaken the gate<br/><i>disable plugin · exclude class ·<br/>lower threshold · add suppression</i>"| W["Requires an <b>ADR</b><br/>plus the user's agreement<br/><br/><i>never a quiet edit to the build file</i>"]
    ROUTE -->|"change an existing test<br/><i>delete assertion · loosen matcher ·<br/>disable · widen tolerance ·<br/>exclude from coverage</i>"| T["<b>Tell the user first</b><br/>cause · reading · proposal · alternative<br/><br/><i>default assumption: the new code is wrong,<br/>not the test</i>"]

    F --> AGAIN
    W --> AGAIN
    T --> AGAIN(["Run the build again —<br/><i>phase 6 is not done until it is green</i>"])

    style RED fill:#c0392b,color:#fff
    style G fill:#27ae60,color:#fff
    style F fill:#27ae60,color:#fff
    style W fill:#f9e79f,stroke:#b7950b
    style T fill:#f9e79f,stroke:#b7950b
```

Coverage behaves as a ratchet rather than a cliff:

```mermaid
---
config:
  layout: elk
  elk:
    mergeEdges: true
    nodePlacementStrategy: BRANDES_KOEPF
---
flowchart LR
    M["Measure current coverage"] --> C{"At or above<br/>the 80% floor?"}
    C -->|yes| T["Gate at 80% floor<br/>~90% on new / changed code"]
    C -->|no| S["Gate set to the <b>current</b> number,<br/>and that number is reported"]
    S --> N["New / changed code still meets ~90%"]
    N --> U["Gate may <b>never</b> decrease"]
    U -->|"lifting toward 80%<br/>is its own ticket"| U
    T --> U
```

---

## 6. The decision rules behind the tool choices

*Take away: precedence, the open-source bar and the copyleft split resolve most tooling questions without opening a reference file.*

```mermaid
---
config:
  layout: elk
  elk:
    mergeEdges: true
    nodePlacementStrategy: BRANDES_KOEPF
---
flowchart TD
    subgraph DEP["Adding a library"]
        D1{"On the platform surface<br/>or a named default?"}
        D1 -->|yes| D2["Already approved"]
        D1 -->|no| D3["<b>A proposal</b> — name what it buys<br/>and what it replaces, then WAIT"]
    end

    subgraph LIC["Licensing — one distinction decides it"]
        L1{"Does it ship inside<br/>the artifact?"}
        L1 -->|"build / test / CI tooling"| L2["License governs the tool,<br/>not your output — <b>fine, incl. copyleft</b>"]
        L1 -->|"runtime dependency"| L3{"GPL / AGPL / LGPL?"}
        L3 -->|yes| L4["<b>Blocker</b> — report it,<br/>never resolve it alone"]
        L3 -->|no| L5["Permissive — unproblematic"]
    end

    subgraph BAR["The bar every tool must clear"]
        B1{"Open source, runnable locally<br/>natively or in Podman,<br/>no subscription / seat / cloud account?"}
        B1 -->|no| B2["Not a default —<br/>propose a free alternative<br/><i>Docker Desktop · Flyway Teams ·<br/>hosted siblings of free tools</i>"]
        B1 -->|yes| B3["Usable"]
        B4(["GitHub — the single<br/>accepted hosted exception"]) -.-> B3
    end

    subgraph PREC["Precedence when sources disagree"]
        direction LR
        X1["Project ADRs<br/>+ CLAUDE.md"] --> X2["This skill"] --> X3["Your own default"]
    end


    style B2 fill:#f9e79f,stroke:#b7950b
    style L4 fill:#c0392b,color:#fff
    style D3 fill:#8e44ad,color:#fff
```

---

## 7. The ideas in one picture

*Take away: five convictions generate everything above.*

```mermaid
mindmap
  root(("Disciplined Delivery"))
    ("Quality is the constraint")
      ("Speed comes from not redoing work")
      ("A gate that only warns is not a gate")
      ("The same gates run locally and in CI")
    ("No invisible decisions")
      ("Every choice traces to a criterion, ADR, NFR or answer")
      ("A reasonable default is still a decision")
      ("Skipping a phase silently is itself an invisible decision")
    ("No assumptions")
      ("Ask the moment something is unclear, not later")
      ("Mirror the topic back before defining anything")
      ("Silence is an assumption, an explicit none is a decision")
    ("Definition before code")
      ("The ticket is the source of truth, not the chat")
      ("Plan committed before implementing")
      ("Contract agreed before implementation")
      ("Failing test before the fix")
    ("Nothing durable lives in a session")
      ("Ticket, plan file, ADR, commits")
      ("A manual step done twice becomes a script")
      ("A release is never hand-typed")
      ("Screenshots regenerate instead of rotting")
```

> `mindmap` is a newer Mermaid diagram type and GitHub can lag Mermaid releases — verify it renders with one throwaway commit before relying on it, or fall back to a `flowchart LR`.
