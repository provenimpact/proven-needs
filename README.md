# proven-needs

Intent-driven state transition workflow for evolving software systems.

Declare a **desired state**, evaluate it against **reality** and **constraints**, then execute the **minimal valid transition** to make it true. Both feature work and maintenance use the same mechanism.

## How It Works

```mermaid
flowchart TD
    START((User declares<br/>desired state)) --> OBSERVE

    subgraph LOOP ["State Transition Loop"]
        OBSERVE["1. Observe<br/><i>Capture current state:<br/>artifacts, codebase, deps, security</i>"]
        DECLARE["2. Declare<br/><i>Classify intent &amp; decompose<br/>into features</i>"]
        EVALUATE["3. Evaluate<br/><i>Check feasibility, constraints,<br/>staleness</i>"]
        DERIVE["4. Derive<br/><i>Build minimal transition plan<br/>from capability graph</i>"]
        EXECUTE["5. Execute<br/><i>Invoke capabilities in<br/>dependency order</i>"]
        VALIDATE["6. Validate<br/><i>Verify desired state is true,<br/>constraints hold</i>"]

        OBSERVE --> DECLARE
        DECLARE --> EVALUATE
        EVALUATE -->|Feasible| DERIVE
        EVALUATE -->|Violation| BLOCK
        DERIVE --> EXECUTE
        EXECUTE --> VALIDATE
        VALIDATE -->|Achieved| LOG
        VALIDATE -->|Not achieved| OBSERVE
    end

    BLOCK["Constraint Violation<br/><i>Revise, update constraint,<br/>or abort</i>"] -->|Revised| EVALUATE
    LOG["Record in<br/>state-log.adoc"] --> NEXT
    NEXT((Declare next<br/>desired state)) --> OBSERVE

    style START fill:#4CAF50,color:#fff,stroke:none
    style NEXT fill:#4CAF50,color:#fff,stroke:none
    style BLOCK fill:#f44336,color:#fff,stroke:none
    style LOG fill:#2196F3,color:#fff,stroke:none
```

The system figures out what needs to happen. You declare what must be true.

## Entry Point

Load the `proven-needs` skill. It is the single orchestrator that accepts intents, classifies them, and invokes the appropriate capabilities.

```
I want users to be able to browse products, add them to cart, and checkout
```

The orchestrator will: decompose into features, confirm grouping, then for each feature create stories, derive specs, design, plan tasks, generate tests, implement -- resolving divergences and recording ADRs along the way.

## Capabilities

```mermaid
flowchart LR
    subgraph feature ["Feature Pipeline (per feature package)"]
        direction LR
        STORIES["needs-stories<br/><i>WHY</i>"]
        SPEC["needs-spec<br/><i>WHAT</i>"]
        DESIGN["needs-design<br/><i>HOW</i>"]
        TASKS["needs-tasks<br/><i>WORK</i>"]
        TESTS["needs-tests<br/><i>VERIFY</i>"]
        IMPL["needs-implementation<br/><i>CODE</i>"]

        STORIES --> SPEC
        STORIES --> DESIGN
        SPEC --> DESIGN
        DESIGN --> TASKS
        STORIES -.->|fallback| TASKS
        SPEC --> TESTS
        DESIGN -.->|optional| TESTS
        TASKS --> IMPL
        TESTS --> IMPL
        DESIGN -.->|fallback| IMPL
    end

    subgraph project ["Project-Wide (independent)"]
        direction LR
        ADR["needs-adr"]
        ARCH["needs-architecture"]
        DEPS["needs-dependencies"]
        SEC["needs-security"]
        COMP["needs-compliance"]
    end

    INTENT((Orchestrator<br/>proven-needs)) --> feature
    INTENT --> project

    DESIGN -.->|tech decisions| ADR
    SEC -.->|delegates fixes| DEPS
    IMPL -.->|divergences| INTENT
    INTENT -.->|update design| DESIGN
    INTENT -.->|fix code| IMPL

    CONSTRAINTS[("docs/constraints.adoc<br/><i>Checked at every step</i>")] -.->|enforced| feature
    CONSTRAINTS -.->|enforced| project

    style INTENT fill:#4CAF50,color:#fff,stroke:none
    style CONSTRAINTS fill:#FF9800,color:#fff,stroke:none
```

**Solid arrows** = required dependency. **Dotted** = optional/fallback.

| Scope | Capability | Skill | Domain |
|---|---|---|---|
| Feature | Stories | `needs-stories` | WHY: user needs |
| Feature | Specs | `needs-spec` | WHAT: black-box requirements |
| Feature | Design | `needs-design` | HOW: implementation blueprint |
| Feature | Tasks | `needs-tasks` | WORK: phased coding units |
| Feature | Tests | `needs-tests` | VERIFY: spec-derived test cases |
| Feature | Implementation | `needs-implementation` | CODE: working software |
| Project | ADRs | `needs-adr` | Technology decisions |
| Project | Architecture | `needs-architecture` | System structure (C4 model) |
| Project | Dependencies | `needs-dependencies` | Dependency management |
| Project | Security | `needs-security` | Security posture |
| Project | Compliance | `needs-compliance` | License/policy compliance |
| Support | EARS | `ears-requirements` | Requirement syntax reference |

Every capability follows **observe/evaluate/execute**: assess current state, check if action is needed and constraints allow it, make minimum changes.

## Artifact Traceability

```mermaid
flowchart LR
    S["user-stories.adoc<br/>:version:"]
    SP["spec.adoc<br/>:source-stories-version:"]
    D["design.adoc<br/>:source-stories-version:<br/>:source-spec-version:"]
    T["tasks.adoc<br/>:source-design-version:<br/>:source-stories-version:<br/>:source-spec-version:"]

    S -->|tracked by| SP
    S -->|tracked by| D
    SP -->|tracked by| D
    S -->|tracked by| T
    SP -->|tracked by| T
    D -->|tracked by| T

    style S fill:#4CAF50,color:#fff,stroke:none
    style SP fill:#2196F3,color:#fff,stroke:none
    style D fill:#FF9800,color:#fff,stroke:none
    style T fill:#9C27B0,color:#fff,stroke:none
```

Each downstream artifact tracks its upstream version. When an upstream changes, downstream artifacts become stale. The orchestrator detects cascades and includes sync steps in the transition plan.

| Capability | Reads | Writes |
|---|---|---|
| `needs-stories` | `docs/constraints.adoc` | `user-stories.adoc` |
| `needs-spec` | `user-stories.adoc`, `docs/constraints.adoc` | `spec.adoc` |
| `needs-design` | `user-stories.adoc`, `spec.adoc`, ADRs, `docs/constraints.adoc`, `architecture.adoc` | `design.adoc`, `data-model.adoc`, `contracts/` |
| `needs-tasks` | `design.adoc` (or `user-stories.adoc` fallback), `spec.adoc`, `docs/constraints.adoc` | `tasks.adoc` |
| `needs-implementation` | `tasks.adoc` (or `design.adoc` fallback), `user-stories.adoc`, `spec.adoc`, `docs/constraints.adoc`, ADRs | source code |
| `needs-tests` | `spec.adoc`, `design.adoc`, `user-stories.adoc`, `docs/constraints.adoc`, source code | test files |
| `needs-adr` | existing ADRs | `docs/adrs/*.adoc`, `index.adoc` |
| `needs-architecture` | all feature designs, ADRs, `docs/constraints.adoc`, codebase | `docs/architecture.adoc` |
| `needs-dependencies` | package manifests, `docs/constraints.adoc` | package manifests, lockfiles |
| `needs-security` | codebase, dependencies, config, `docs/constraints.adoc` | source code, config |
| `needs-compliance` | dependencies, `docs/constraints.adoc` | dependencies, `docs/constraints.adoc` |

## Visual Reference

### Intent Classification

```mermaid
flowchart TD
    INPUT["User intent"] --> SIGNALS{"Analyze signals"}

    SIGNALS -->|"User journey"| FEAT["Feature evolution"]
    SIGNALS -->|"Universal quantifiers"| CONST["Constraint declaration"]
    SIGNALS -->|"Sync/update language"| ART["Artifact maintenance"]
    SIGNALS -->|"Packages, vulns"| DEP["Dependency maintenance"]
    SIGNALS -->|"System structure"| ARCH["Architecture evolution"]
    SIGNALS -->|"Tests, coverage"| QUAL["Quality improvement"]
    SIGNALS -->|"Docs, architecture"| DOC["Documentation"]
```

### Constraint Detection

```mermaid
flowchart TD
    INTENT["Intent"] --> Q1{"Universal scope?"}
    Q1 -->|Yes| Q2{"System-as-subject?"}
    Q1 -->|No| FEATURE["Feature requirement"]
    Q2 -->|Yes| Q3{"No user journey?"}
    Q2 -->|No| FEATURE
    Q3 -->|Yes| Q4{"Future-proof?"}
    Q3 -->|No| ASK["Ask user"]
    Q4 -->|Yes| CONSTRAINT["Constraint"]
    Q4 -->|No| ASK
```

### Feature Decomposition (Two-Pass)

```mermaid
flowchart TD
    START((Intent)) --> CHECK{Existing features?}
    CHECK -->|No: Greenfield| GF_P1
    CHECK -->|Yes: Evolution| EV_P1

    subgraph greenfield ["Greenfield"]
        GF_P1["Pass 1: Draft stories into _drafts/"]
        GF_P1 --> GF_COHESION["Analyze cohesion"] --> GF_PROPOSE["Propose groupings"]
        GF_PROPOSE --> GF_CONFIRM{Confirm?}
        GF_CONFIRM -->|Yes| GF_P2["Pass 2: Distribute to packages"]
        GF_CONFIRM -->|Adjust| GF_PROPOSE
    end

    subgraph evolution ["Evolution"]
        EV_P1["Pass 1: Draft + classify against existing"]
        EV_P1 --> EV_PROPOSE["Present mapping"]
        EV_PROPOSE --> EV_CONFIRM{Confirm?}
        EV_CONFIRM -->|Yes| EV_P2["Pass 2: Distribute"]
        EV_CONFIRM -->|Adjust| EV_PROPOSE
    end

    GF_P2 --> DONE((Ready))
    EV_P2 --> DONE
```

### Risk Classification

```mermaid
flowchart TD
    CHANGE["Transition"] --> FACTORS["Assess: scope, constraints, reversibility, code impact"]
    FACTORS --> LOW["Low risk → auto-approve"]
    FACTORS --> MED["Medium → propose, ask"]
    FACTORS --> HIGH["High → full plan, require approval"]
```

### Design Divergence Resolution

```mermaid
sequenceDiagram
    participant Impl as needs-implementation
    participant Orch as Orchestrator
    participant User as User
    participant Design as needs-design

    Impl->>Orch: Report divergences
    Orch->>User: Present with analysis

    loop Each divergence
        User->>Orch: Choose resolution
        alt Update design
            Orch->>Design: Reconciliation mode
        else Fix code
            Orch->>Impl: Fix divergence
        end
    end
```

### Feature Status

```mermaid
stateDiagram-v2
    [*] --> Stories : user-stories.adoc
    Stories --> Specified : spec.adoc
    Specified --> Designed : design.adoc (Current)
    Designed --> Planned : tasks.adoc (Current)
    Planned --> Tested : tests from spec
    Tested --> Implemented : all tests pass

    Stories --> Archived
    Specified --> Archived
    Designed --> Archived
    Planned --> Archived
    Tested --> Archived
    Implemented --> Archived
    Archived --> Stories : un-archived
```

## What This Is Not

- **Not waterfall** -- the desired state is disposable; declare a new one each iteration
- **Not backlog-driven** -- work is derived each iteration, not accumulated in a backlog
- **Not uncontrolled** -- constraints anchor behavior and prevent regressions

## Reference

- [EARS Quick Reference](skills/ears-requirements/references/ears-reference.adoc) -- Requirement syntax standard
- [Example Session](skills/proven-needs/references/example-session.adoc) -- Full walkthrough of feature and maintenance intents
