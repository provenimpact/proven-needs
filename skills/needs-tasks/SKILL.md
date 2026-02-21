---
name: needs-tasks
description: Create phased implementation task lists from design documents. Use when asked to create tasks, plan implementation, break down the design into work items, create an implementation backlog, or prepare for coding. Reads docs/design/, user-stories.adoc, and docs/specs/ to produce a tickable task list in docs/tasks/ organized into sequential phases with parallelism markers and full traceability back to specs and stories. This is the final step in the proven-needs pipeline before implementation begins.
---

## Prerequisites

Load the `proven-needs` skill first. It provides the overall workflow context, artifact locations, and conventions.

## Workflow

Creating an implementation task list involves these steps:

1. Read inputs
2. Check for existing task list
3. Analyze design for task decomposition
4. Organize into phases
5. Write task list
6. Quality checklist

### 1. Read Inputs

Read these sources in order:

1. **`docs/design/design.adoc`** -- primary driver. Extract `:version:`, `:status:`, system design sections, and story resolution mappings. Also read `data-model.adoc` and `contracts/` if they exist. **If missing:** Stop and inform the user that a design must be created first using the `needs-design` skill.

2. **If `:status:` is `Stale`:** Warn the user that the design is stale. Ask whether to proceed anyway or update the design first.

3. **`user-stories.adoc`** -- for traceability. Extract story IDs and titles.

4. **`docs/specs/`** -- for traceability. Extract all spec IDs and their categories.

5. **Existing codebase** -- if this is not a greenfield project, analyze what already exists to avoid creating tasks for things already implemented.

### 2. Check for Existing Task List

Look for `docs/tasks/tasks.adoc`.

**If no task list exists:** Continue with step 3.

**If task list exists:** Read `:status:` and `:source-design-version:`.

| Condition | Action |
|---|---|
| `:status:` is `Implemented` | Previous task list is complete. Start a fresh task list (overwrite). |
| Source version matches current design and no tasks are ticked | Inform user task list appears current. Ask to force recreate or stop. |
| Source version matches but some tasks are ticked | Inform user of progress. Ask to continue with existing list or recreate (warn about losing progress). |
| Source version differs from current design | Task list is stale. Present summary of what changed in the design. Ask whether to recreate from scratch or incrementally update (preserving ticked tasks where possible). |

### 3. Analyze Design for Task Decomposition

Walk through the design document systematically to identify discrete implementation units. Each task should be a single coding unit -- small enough to implement in one sitting.

**Sources of tasks:**

| Design Section | Task Types |
|---|---|
| Data model entities | Entity/model creation, migrations, validation rules |
| System design components | Module/service setup, core logic implementation |
| Interface contracts / API endpoints | Endpoint implementation, request/response handling |
| Frontend components | Component creation, state management wiring |
| External integrations | Service client setup, integration logic |
| Story resolution -- error cases | Error handling, edge cases |
| Story resolution -- notifications | Notification/email implementation |

**For each identified task, record:**
- A clear, actionable title
- Which design components are involved
- Which spec IDs it satisfies
- Which user stories it contributes to
- Whether it depends on other tasks (determines phase placement and parallelism)

### 4. Organize into Phases

Group tasks into sequential phases based on implementation dependencies. Phases are completed in order -- all tasks in a phase should be completable before moving to the next phase.

**Typical phase progression** (adapt to the project):

| Phase | Purpose | Examples |
|---|---|---|
| Foundation | Infrastructure, data model, configuration | Database setup, entity creation, project scaffolding |
| Core Logic | Business logic, services, core modules | Service implementations, domain logic |
| Interface Layer | APIs, UI components, CLI commands | Endpoint handlers, React components, command parsers |
| Integration | External services, cross-cutting concerns | Payment providers, email services, auth |
| Polish | Error handling, edge cases, notifications | Error responses, validation messages, notification triggers |

Within each phase, mark every task:

- **`[parallel]`** -- can be implemented concurrently with other parallel tasks in the same phase. No dependency on other tasks within the phase.
- **`[sequential]`** -- must be completed before subsequent sequential tasks in the same phase. Has intra-phase dependencies.

**Guidelines for phase assignment:**
- A task belongs in the earliest phase where all its dependencies are satisfied
- Prefer more parallel tasks over fewer sequential ones -- split tasks to enable parallelism where possible
- If a phase would contain only one task, consider merging it with an adjacent phase

### 5. Write Task List

Create `docs/tasks/` directory and write `docs/tasks/tasks.adoc`:

```asciidoc
= Implementation Tasks
:version: 1.0.0
:status: Current
:source-design-version: <design version>
:source-stories-version: <user-stories version>
:source-specs-version: <specs version>
:last-updated: YYYY-MM-DD
:toc:

== Overview

<Brief summary: what is being implemented, number of phases, total number of tasks.>

== Phase 1: <Phase Name>

<One-sentence phase purpose.>

* [ ] TASK-001: <Task title> [parallel]
+
Components:: <design components involved>
Stories:: <story numbers>
Specs:: <spec IDs>
Description:: <What to implement and key details>

* [ ] TASK-002: <Task title> [sequential]
+
Components:: <design components involved>
Stories:: <story numbers>
Specs:: <spec IDs>
Description:: <What to implement and key details>

== Phase 2: <Phase Name>

...

== Traceability Matrix

[cols="1,1,1", options="header"]
|===
| Story | Spec IDs | Tasks

| Story 1: <title> | SPEC-001, SPEC-002 | TASK-001, TASK-005
| Story 2: <title> | SPEC-003 | TASK-002, TASK-006
|===
```

**`:status:` values:**
- `Current` -- task list is valid and aligned with design
- `Stale` -- design has changed since this task list was created
- `Implemented` -- all tasks are ticked off

**Version rules:**
- `:version:` uses SemVer, starts at `1.0.0`
- `:source-design-version:` records which design version was used
- `:source-stories-version:` and `:source-specs-version:` record upstream versions for full traceability
- `:last-updated:` set to today's date

**Task IDs:** Sequential within the document: TASK-001, TASK-002, etc. IDs are stable -- do not renumber when updating.

**Ticking off tasks:** When a task is completed, change `[ ]` to `[x]`. When all tasks in the document are ticked, set `:status:` to `Implemented`.

### 6. Quality Checklist

Before finalizing, verify every item:
- Every spec ID from `docs/specs/` appears in at least one task
- Every user story is covered by the aggregate tasks
- Every design section (system design, data model, contracts) has corresponding tasks
- No circular dependencies exist between phases
- Phase ordering respects actual implementation dependencies
- Each task is a discrete, implementable coding unit
- Parallel/sequential markers are correct (parallel tasks truly have no intra-phase dependencies)
- The traceability matrix is complete and accurate
- Source versions are recorded correctly

## Reference

See `references/example.adoc` for a complete example showing how a design document becomes a phased task list with traceability, continuing the e-commerce project used in the `needs-design` example.
