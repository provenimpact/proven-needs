---
name: needs-tasks
description: Create phased implementation task lists for a feature. Use when the proven-needs orchestrator determines that a feature needs a task breakdown. Operates within a single feature package at docs/features/<slug>/. Tasks define WORK — discrete coding units organized into sequential phases with parallelism markers and traceability.
---

## Prerequisites

Invoked by the `proven-needs` orchestrator with feature context (slug, intent, current state).

## Observe

1. Read `docs/features/<slug>/design.adoc`: extract `:version:`, `:status:`, system design, story resolution. Also read `data-model.adoc` and `contracts/` if present. If missing, tasks will derive from stories directly (set `:source-design-version: n/a`).
2. Read `user-stories.adoc` (story IDs, titles, criteria, `:version:`) and `spec.adoc` (spec IDs, `:version:`). If spec missing, set `:source-spec-version: n/a`.
3. If `tasks.adoc` exists: read `:version:`, `:status:`, source versions, phases, tasks, tick states.
4. Read `docs/constraints.adoc` for quality constraints relevant to task planning.
5. Analyze codebase to avoid creating tasks for things already implemented.

## Evaluate

| Condition | Action |
|---|---|
| No task list exists | Create |
| `:status: Implemented` | Fresh task list (overwrite) |
| Source versions match, no tasks ticked | Current, report to orchestrator |
| Source versions match, some ticked | Partial progress, report |
| `:source-design-version:` differs | Stale -- recommend updating design first (staleness flows through design) |

Do not independently compare `:source-stories-version:` or `:source-spec-version:` -- those are for traceability. Staleness detection flows through the design.

## Execute

### Design-driven task decomposition (default)

Walk through the design systematically. Each task is a single coding unit.

| Design Section | Task Types |
|---|---|
| Data model entities | Entity creation, migrations, validation |
| System design components | Module/service setup, core logic |
| Interface contracts / APIs | Endpoint implementation, request/response |
| Frontend components | Component creation, state wiring |
| External integrations | Service clients, integration logic |
| Story resolution -- errors | Error handling, edge cases |

For each task record: title, design components involved, spec IDs satisfied, user stories contributed to, dependencies (determines phase/parallelism).

### Story-driven task derivation (no design)

Derive tasks from stories and acceptance criteria. Group related criteria into a single task when tightly coupled; split when independently implementable. Omit `Components::` field. Use story groupings to inform phase organization.

### Organize into phases

Group tasks by implementation dependencies:

| Phase | Purpose | Examples |
|---|---|---|
| Foundation | Infrastructure, data model, config | DB setup, entity creation, scaffolding |
| Core Logic | Business logic, services | Service implementations, domain logic |
| Interface Layer | APIs, UI, CLI | Endpoints, React components, parsers |
| Integration | External services, cross-cutting | Payment, email, auth |
| Polish | Error handling, edge cases | Error responses, validation messages |

Within each phase mark tasks: **`[parallel]`** (no intra-phase dependencies) or **`[sequential]`** (must precede subsequent sequential tasks).

Guidelines: earliest phase where dependencies are satisfied, prefer parallel over sequential, merge single-task phases with adjacent ones.

### Write task list

Create `docs/features/<slug>/tasks.adoc`:

```asciidoc
= Implementation Tasks: <Feature Name>
:version: 1.0.0
:status: Current
:source-design-version: <design version>
:source-stories-version: <user-stories version>
:source-spec-version: <spec version>
:last-updated: YYYY-MM-DD
:feature: <slug>
:toc:

== Overview
<Summary: what is being implemented, phases, total tasks.>

== Phase 1: <Phase Name>
<One-sentence purpose.>

* [ ] TASK-001: <Title> [parallel]
+
Components:: <design components>
Stories:: <story numbers>
Specs:: <spec IDs>
Description:: <What to implement>

== Phase 2: <Phase Name>
...

== Traceability
[cols="1,1,1", options="header"]
|===
| Story | Spec IDs | Tasks
| US-001: <title> | PROD-001, PROD-002 | TASK-001, TASK-005
|===
```

**`:status:` values:** `Current`, `Stale`, `Implemented` (all ticked).

**Task IDs:** Sequential (TASK-001, TASK-002, ...). Stable -- do not renumber.

**Ticking:** `[ ]` → `[x]` on completion. All ticked → `:status: Implemented`.

**Lifecycle:** Tasks guide implementation but are not the source of truth for completion -- spec-derived tests serve that role. May be removed after reaching `Implemented`.

## Quality Checklist

- Every spec ID appears in at least one task (skip if `:source-spec-version: n/a`)
- Every user story covered
- Every design section has tasks (skip if `:source-design-version: n/a`)
- No circular phase dependencies
- Each task is a discrete coding unit
- Parallel/sequential markers correct
- Traceability complete
- Source versions recorded

## Reference

See `references/example.adoc` for a complete example.
