---
name: proven-needs
description: Intent-driven state transition workflow for evolving software systems. Declare a desired state, evaluate it against reality and constraints, then execute the minimal valid transition. Use when asked to implement a feature, fix something, update dependencies, improve quality, or make any change to the system. This is the single entry point — it observes current state, classifies the intent, evaluates feasibility against constraints, derives a transition plan, and orchestrates the appropriate needs-* capabilities. Also use when asked about the development workflow, how features are organized, or the overall process.
---

## Purpose

Continuously evolve a software system by declaring a desired state, evaluating it against the current state and constraints, then executing the minimal valid transition to make it true. Both maintenance and feature work are state changes, not task accumulation.

## State Transition Loop

```
Observe → Declare → Evaluate → Derive → Execute → Validate → Repeat
```

1. **Observe** -- capture the current state (automated)
2. **Declare** -- accept a desired state (from user or system-proposed)
3. **Evaluate** -- test feasibility against current state and constraints
4. **Derive** -- determine the minimal transition plan
5. **Execute** -- invoke capabilities to apply changes
6. **Validate** -- verify the desired state is now true
7. **Repeat** -- declare the next desired state

## Core Concepts

### Current State

The observable, verifiable reality of the system right now. Computed fresh each invocation, never stored.

**Artifact state:** which feature packages exist in `docs/features/`, their artifacts, versions, statuses, and staleness. Project-wide artifacts: `docs/constraints.adoc`, `docs/adrs/`, `docs/architecture.adoc`, `docs/state-log.adoc`.

**Codebase state:** language, framework, project structure, dependency graph and versions, test coverage, lint/build status, security posture.

### Desired State

A declarative statement of what must be true after this transition. Sources: the user (explicit intent) or the system (detected conditions, proposed to user).

Examples: "Users can reset their password via SMS" (feature), "All dependencies have no known critical vulnerabilities" (maintenance), "All API endpoints enforce rate limiting" (constraint).

### Constraints (Invariants)

Rules that must not be violated across any transition. Defined in `docs/constraints.adoc`. See `references/constraints-template.adoc` for the file format and example categories.

### Feature Package

A self-contained unit of work scoped to one feature at `docs/features/<slug>/`:

```
docs/features/<slug>/
├── user-stories.adoc    # WHY: user needs and motivations
├── spec.adoc            # WHAT: testable requirements for this feature
├── design.adoc          # HOW: implementation blueprint
└── tasks.adoc           # WORK: phased implementation breakdown
```

Each feature package is fully independent -- no cross-feature dependencies. Feature designs reference project-wide ADRs and architecture but never other feature designs.

### State Log

An append-only audit trail of all state transitions at `docs/state-log.adoc`. See `references/state-log-template.adoc` for format.

## Capabilities

The orchestrator does not produce artifacts directly. It invokes capabilities (the `needs-*` skills) to perform work. Each capability follows the observe/evaluate/execute pattern.

### Invoking a capability

**Load each capability skill by name** using the skill-loading tool before performing its work. Do NOT attempt to produce artifacts without loading the skill definition first. The orchestrator plans and coordinates; capability skills contain the detailed artifact instructions.

Invocation steps:
1. Load the skill by name (e.g., `needs-stories`, `needs-spec`, `needs-design`)
2. The capability runs its own observe → evaluate → execute cycle
3. Wait for completion before proceeding to the next capability
4. If a capability references another skill (e.g., `needs-design` may load `needs-adr`), that skill must also be loaded

### Feature-scoped capabilities

| Capability | Skill | Domain |
|---|---|---|
| Stories | `needs-stories` | Create/update user stories for a feature |
| Specifications | `needs-spec` | Derive testable requirements from stories |
| Design | `needs-design` | Create implementation blueprint for a feature |
| Tasks | `needs-tasks` | Break design into phased implementation units |
| Tests | `needs-tests` | Derive and generate tests from specifications |
| Implementation | `needs-implementation` | Write and verify code for a feature |

### Project-wide capabilities

| Capability | Skill | Domain |
|---|---|---|
| ADRs | `needs-adr` | Record technology decisions |
| Architecture | `needs-architecture` | Document current system architecture |
| Dependencies | `needs-dependencies` | Manage and update dependency graph |
| Security | `needs-security` | Assess and remediate security posture |
| Compliance | `needs-compliance` | Verify license and policy compliance |

### Supporting skills

| Skill | Purpose |
|---|---|
| `ears-requirements` | EARS methodology reference for stories and specs |

## Workflow

### 1. Observe Current State

When this skill is invoked, immediately build the current state model:

#### 1.1 Read project-wide artifacts

1. **`docs/constraints.adoc`** -- read all constraint categories and rules. If missing, note it. Do not create it automatically -- constraints are declared intentionally.

2. **`docs/features/`** -- list all feature directories. For each, check which artifacts exist and read their `:version:` and `:status:` attributes. Features with `:status: Archived` are reported but skipped during intent classification and staleness checks.

3. **`docs/adrs/`** -- read the index, note counts and statuses.

4. **`docs/architecture.adoc`** -- check existence, read `:version:` if present.

5. **`docs/state-log.adoc`** -- check existence, read recent transitions. Pay particular attention to:
   - **`:result: In Progress`** -- prior session ended unexpectedly. Propose resuming or marking as Failed before starting new work.
   - **`:result: Partial`** -- user explicitly stopped mid-transition. Propose completing remaining capabilities before starting new work.

#### 1.2 Analyze codebase

1. **Project type** -- detect language, framework, build system from config files
2. **Dependencies** -- parse dependency files for outdated packages, vulnerabilities, license info
3. **Quality signals** -- build, lint, test status and coverage
4. **Code structure** -- directory layout, module organization, existing patterns

#### 1.3 Present state summary

```
Current state:
  Features: 3 (user-auth [implemented], user-profile [designed], shopping-cart [stories only])
  Constraints: 8 rules across 4 categories
  ADRs: 2 accepted
  Architecture: v1.0.0 (current)
  Codebase: TypeScript/Next.js, 47 deps (1 vulnerable), 78% coverage, build passing
  Constraint violations: specs stale in user-profile (stories updated since spec)
```

### 2. Accept Desired State

The user states what they want to be true. The orchestrator interprets this as a desired state.

#### 2.1 Intent classification

Classify the desired state into one or more intent types:

| Intent Type | Signals | Example |
|---|---|---|
| **Feature evolution** | User-facing capability, user journey | "Users can reset password via SMS" |
| **Constraint declaration** | Universal quantifiers, system-as-subject, future-proof | "All API endpoints must enforce rate limiting" |
| **Artifact maintenance** | References existing artifacts, sync/update language | "Specs are in sync with current stories" |
| **Dependency maintenance** | References packages, versions, vulnerabilities | "No dependencies have known vulnerabilities" |
| **Architecture evolution** | References system structure, technology changes | "Authentication uses OAuth2 instead of sessions" |
| **Quality improvement** | References tests, coverage, code quality | "All API endpoints have integration tests" |
| **Documentation** | References docs, architecture document | "Architecture doc reflects current system" |

#### 2.2 Constraint detection

Before feature decomposition, check whether the intent is actually a constraint. An intent is a constraint if it has all four properties:

1. **Universal scope** -- quantifiers like "all", "every", "no X may", "must always", "never"
2. **System-as-subject** -- describes a system property, not a user capability
3. **No user journey** -- no identifiable user role, action, or benefit
4. **Future-proof** -- applies to features that don't exist yet

If the intent is a constraint: propose adding it to `docs/constraints.adoc`, ask for confirmation, update the file and state log. Do not create a feature package.

If uncertain, ask: "Is this a project-wide constraint (enforced everywhere, current and future) or a feature-specific requirement?"

#### 2.3 Feature decomposition (for feature evolution intents)

Uses a two-pass approach because `needs-stories` operates within a feature package (requires a slug), but feature groupings aren't known until stories are drafted.

**Pass 1 -- Draft stories with a temporary slug:**

1. Invoke `needs-stories` with slug `_drafts` to derive user stories from the intent
2. Analyze story cohesion to propose feature groupings:
   - Stories sharing data entities → same feature
   - Stories in the same user journey → same feature
   - Stories delivering independent value → separate features
3. **When features already exist:** classify each drafted story against existing features (extends / new / updates) using keyword matching, data entity overlap, and user journey alignment
4. Present the proposed grouping to the user and wait for confirmation

**Pass 2 -- Distribute stories into feature packages:**

5. For each confirmed feature, invoke `needs-stories` with the final slug. Story IDs are reassigned sequentially within each feature (US-001, US-002, ...).
6. Remove the temporary `_drafts` directory

**Constraint surfacing during decomposition:**

If a requirement is identified as cross-cutting while deriving stories/specs, flag it as a potential constraint and ask the user whether to add it to `docs/constraints.adoc` (recommended) or keep it as a feature spec.

### 3. Evaluate Feasibility

For each feature in the transition plan:

#### 3.1 Precondition check

Does the desired state require artifacts that don't exist yet? Capability dependencies:
- `needs-spec` requires stories
- `needs-design` requires stories and spec
- `needs-tasks` works best with design
- `needs-implementation` requires at minimum a design
- `needs-tests` requires spec

If preconditions are unmet, satisfy them by invoking earlier capabilities first. This is not a rigid pipeline -- the orchestrator dynamically determines what's needed.

#### 3.2 Constraint check

Test the proposed transition against all constraints in `docs/constraints.adoc`. If a constraint would be violated, present options: revise the approach, update the constraint (requires justification), or abort.

#### 3.3 Staleness check

Check if existing artifacts involved in the transition are stale (stories updated but spec not synced, spec updated but design not refreshed, etc.). Recommend resolution before proceeding.

### 4. Derive Transition Plan

Build a dependency graph of capability invocations. The graph is derived, not hardcoded.

**For each feature in scope:**

1. Determine which artifacts need creating or updating
2. Order capabilities by dependency: stories → specs → design → tasks → tests → implementation. `needs-spec` is always invoked -- every feature gets a specification. `needs-tests` runs before `needs-implementation` -- tests are the acceptance gate for implementation.
3. Skip capabilities whose artifacts are already current
4. Mark which steps can run in parallel across features (independent features can be processed concurrently)

**Architecture updates:** invoke `needs-architecture` after all feature implementations are complete if the system's component structure changed, the architecture document doesn't exist, or is stale.

**Present the plan to the user:**

```
Transition plan to achieve "Users can reset password via SMS":

  Feature: user-authentication/ (extend existing)
  1. needs-stories  2. needs-spec  3. needs-design
  4. needs-tasks    5. needs-tests  6. needs-implementation

  Skipping: needs-adr (no new technology decisions)
  Post-implementation: needs-architecture

  Risk: HIGH | Estimated artifacts affected: 4 files + tests + code
  Proceed?
```

#### Execution mode

After the user approves, ask how to proceed:
- **Autonomous** -- execute all capabilities without pausing
- **Interactive** -- confirm before each capability (default)

### 5. Execute Transition

**Before the first capability**, append an `In Progress` entry to `docs/state-log.adoc` with: `:date:`, `:intent:`, `:type:`, `:risk:`, `:features:`, `:desired-state:`, `:prior-state:`, `:result: In Progress`. Leave `:capabilities-invoked:`, `:constraints-checked:`, `:artifacts-modified:` empty (filled at completion).

Invoke capabilities in derived order by loading each skill. For each:
1. Pass feature context (slug, desired state, current state)
2. Capability runs observe → evaluate → execute
3. Validate output before proceeding

**Between capabilities:** verify artifact correctness, check constraints, update state model.

### Transition progress tracking

Use the todo-list tool to track all capabilities. After each completes, mark it done.

- **Interactive mode:** present updated checklist after each capability, ask to continue
- **Autonomous mode:** proceed immediately, report progress inline

Rules:
- **Do NOT skip capabilities in the plan** unless the user explicitly asks to stop
- **Do NOT treat `needs-implementation` as the final step** -- post-implementation capabilities (architecture, divergence resolution) are part of the plan
- If stopping mid-transition: set `:result: Partial` in the state log entry, list completed and remaining capabilities

**Design divergence resolution (after `needs-implementation`):**

When `needs-implementation` finishes, it reports divergences between the design and what was built. For each divergence it provides: what was specified vs. implemented, analysis of both resolution directions, and rationale.

Present this to the user. For each divergence:
- "update design" → invoke `needs-design` (reconciliation mode)
- "fix code" → re-invoke `needs-implementation` with the specific fix

After `needs-implementation` completes, verify it produced a divergence report (either divergences or explicit "no divergences" confirmation). Request the report if missing.

**Error handling:**
- Capability fails validation → stop, report, ask how to proceed
- Constraint violated → stop, report, offer to revise or abort
- User stops mid-transition → set `:result: Partial` in state log

### 6. Validate

After all capabilities have executed:
1. Re-observe the current state
2. Compare against the original desired state
3. Verify all constraints still hold
4. Run verification commands (build, test, lint) if code was changed

If achieved: update state log entry to `:result: Achieved`. If not: identify what's missing, propose additional steps, leave as `In Progress` or set to `Failed`.

### 7. Record Transition

Update the `In Progress` entry in `docs/state-log.adoc` with the final result: fill in `:capabilities-invoked:`, `:constraints-checked:`, `:artifacts-modified:`, and set `:result:` to `Achieved`, `Partial`, or `Failed`.

## Risk Classification

Transitions are classified by risk:

| Risk | Auto-approve? | Criteria |
|---|---|---|
| **Low** | Yes, execute immediately | Patch deps, metadata fixes, sync with unchanged semantics |
| **Medium** | Propose with summary, ask | Minor deps, design adjustments, adding specs for existing stories |
| **High** | Full plan, require approval | New features, breaking changes, arch changes, major bumps, code |

Risk factors: scope (artifacts affected), constraint proximity, reversibility, code impact.

**System-proposed intents:** The orchestrator can detect conditions and propose desired states (e.g., "Dependency X has a critical CVE"). For auto-approved transitions, inform the user after execution.

## Constraints Specification

File: `docs/constraints.adoc`. See `references/constraints-template.adoc` for the full format and example.

**Lifecycle:**
- **Adding:** classified from intent or surfaced during spec derivation. Requires user confirmation. MINOR bump.
- **Modifying:** user explicitly requests. Requires confirmation. MINOR or MAJOR bump.
- **Removing:** user explicitly requests. Requires confirmation with enforcement-loss warning. MAJOR bump.

A constraint violation blocks a transition unless the user explicitly updates the constraint.

## State Log Specification

File: `docs/state-log.adoc`. See `references/state-log-template.adoc` for the full format and example.

**Conventions:**
- Numbered sequentially (TRANSITION-001, TRANSITION-002, ...), newest first
- Created at execution start with `:result: In Progress`, updated once with final result
- `:result:` values: `In Progress`, `Achieved`, `Partial`, `Failed`

## Feature Package Conventions

### Slug naming

Kebab-case from the feature's primary purpose (e.g., `user-authentication`, `shopping-cart`). Slugs are stable -- do not rename after creation. If scope changes significantly, create a new feature and archive the old one.

### Feature status

Derived from which artifacts exist:

| Artifacts Present | Status |
|---|---|
| user-stories.adoc only | `Stories` |
| + spec.adoc | `Specified` |
| + design.adoc (Current) | `Designed` |
| + tasks.adoc (Current) | `Planned` |
| + test files from spec | `Tested` |
| All spec-derived tests pass | `Implemented` |
| `:status: Archived` in stories | `Archived` |

**Task cleanup:** Once `Implemented`, `tasks.adoc` may be removed -- tests serve as the completion oracle.

### Feature archival

Set `:status: Archived` in `user-stories.adoc`, MAJOR version bump, record in state log. Archived features: skipped in intent classification and staleness checks, remain on disk, can be un-archived.

### Artifact versioning

Each artifact uses SemVer independently: MAJOR (removed/rewritten), MINOR (added/modified), PATCH (typos/metadata). Downstream artifacts track upstream via `:source-*-version:` attributes.

### Format and dates

All artifacts use AsciiDoc (`.adoc`). Dates: `YYYY-MM-DD`. Diagrams: Mermaid in `[source,mermaid]` blocks.

### Diagram conventions

**Architecture documentation** uses C4 model via Mermaid (`C4Context`, `C4Container`, `C4Component`, `C4Deployment`). The `needs-architecture` skill determines which levels to include based on project complexity.

**Feature design documentation** uses Mermaid for component interaction diagrams, sequence diagrams, state diagrams, and data flow diagrams. At minimum one component interaction or sequence diagram per design.

### Requirement syntax

All acceptance criteria and specifications use EARS sentence types. Load the `ears-requirements` skill for the methodology reference.

### Black-box constraint

Feature specifications describe only externally observable behavior. Internal architecture details belong in design, architecture, and ADRs.

## Bootstrap

When this skill is loaded, **immediately** check the project's `AGENTS.md` for the proven-needs workflow marker.

1. Read `AGENTS.md` in the project root (may not exist).
2. Search for `<!-- proven-needs:start -->`.
3. **If found** -- do nothing (already bootstrapped).
4. **If not found** -- check for legacy marker `<!-- proven-needs:start -->`. If found, replace the block. If neither exists, append the block below to the end of the file.

```markdown
<!-- proven-needs:start -->
## Development Workflow
This project uses the proven-needs state transition workflow.
To make changes, declare a desired state and the system will derive
the minimal valid transition: Observe → Evaluate → Derive → Execute → Validate.
Feature work is organized in `docs/features/`. Project constraints are in `docs/constraints.adoc`.
Load the `proven-needs` skill to start.
<!-- proven-needs:end -->
```

5. Inform the user that `AGENTS.md` was updated.

**Rules:** idempotent, append to end, do NOT modify existing block, run before any other workflow task.
