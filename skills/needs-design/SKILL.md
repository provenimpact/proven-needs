---
name: needs-design
description: Create and maintain implementation design documents for a feature. Use when the proven-needs orchestrator determines that a feature needs a design created, updated, or synced with upstream changes. Operates within a single feature package at docs/features/<slug>/. The design document explains HOW — the implementation blueprint constrained by specs and ADRs. Each feature design is independent.
---

## Prerequisites

Invoked by the `proven-needs` orchestrator with feature context (slug, intent, current state).

During Phase 0 (Research), this skill loads `needs-adr` directly when the user confirms a technology decision should be recorded as an ADR.

## Observe

1. Read `docs/features/<slug>/user-stories.adoc`: extract `:version:`, all stories, acceptance criteria. If missing, report to orchestrator.
2. Read `docs/features/<slug>/spec.adoc`: extract `:version:`, all requirement IDs and texts. If missing, note it; design can proceed from stories only (set `:source-spec-version: n/a`).
3. Read project-wide artifacts: `docs/adrs/` (accepted ADRs), `docs/constraints.adoc` (architecture/quality constraints), `docs/architecture.adoc` (current system context).
4. If `docs/features/<slug>/design.adoc` exists: read `:version:`, `:status:`, source versions, full content.
5. Analyze codebase for existing patterns (non-greenfield only).

## Evaluate

| Condition | Action |
|---|---|
| No design exists | Create new design |
| Design exists, source versions match | Report current to orchestrator |
| Design exists, source versions differ | Stale -- sync needed |

Check proposed design against architecture constraints and ADR decisions. Flag conflicts.

## Execute

### Phase 0: Research and Decisions

Analyze stories and specs to identify:

1. **Technology decisions** -- cross-reference against existing ADRs. For each decision not covered by an ADR:
   - Present to user with context, alternatives, and recommendation
   - If user says "create ADR" → load `needs-adr` and create it before proceeding
   - If not → record in design's "Decisions and Constraints" section as unrecorded decision

2. **Unknowns and dependencies** -- resolve with user or list as open questions.

3. **Existing system analysis** (non-greenfield) -- patterns, reusable code, integration points.

Do NOT skip this step. Technology decisions are the primary source of ADRs.

### Phase 1: Design

Design the solution satisfying stories, specs (when available), and constraints. Structure is adaptive to the project type.

#### System design

Describe components, modules, or services. For each: responsibility, interfaces, key details. Include Mermaid diagrams:

- **Component interaction** (`flowchart`) -- required for every design
- **Sequence diagram** (`sequenceDiagram`) -- for multi-component flows
- **State diagram** (`stateDiagram-v2`) -- for stateful entities
- **Data flow** (`flowchart`) -- when data path is non-obvious

Embed in AsciiDoc `[source,mermaid]` blocks. Structure depends on project type (web app: frontend/backend/DB/API; CLI: parser/logic/formatters; library: public API/internals/extensions; etc.).

**Independence rule:** Design must not reference other feature designs. May reference ADRs and architecture.

#### Data model

If the feature involves persistent data: entities, attributes, relationships, validation rules, state transitions. Write to `docs/features/<slug>/data-model.adoc` when non-trivial.

#### Interface contracts

If the feature exposes external interfaces: formats, inputs/outputs, errors. Write to `docs/features/<slug>/contracts/` when relevant.

#### Story resolution

For each user story: which components are involved, how acceptance criteria are met, which spec IDs are satisfied. Every story must appear. When specs available, every spec ID must map to a design element.

### Write design files

Create `docs/features/<slug>/design.adoc`:

```asciidoc
= Design: <Feature Name>
:version: 1.0.0
:status: Current
:source-stories-version: <user-stories version>
:source-spec-version: <spec version>
:last-updated: YYYY-MM-DD
:feature: <slug>
:toc:

== Technical Context
<Project overview. Current state relevant to this feature.>

== Decisions and Constraints
<ADR references. Technology decisions from Phase 0.>

== Research and Unknowns
<Phase 0 findings. Resolved unknowns. Open questions.>

== System Design
<Components, modules, services. Mermaid diagrams.>

== Story Resolution

=== US-001: <Title>
Components:: <involved components>
Criteria::
* <criterion> -> <how design meets it> (SPEC-ID)
```

**`:status:` values:** `Current` (aligned with upstream), `Stale` (upstream changed, sync needed).

### Sync workflow (design already exists)

1. **Quick staleness check** -- compare source versions against current stories/spec versions.
2. **Content analysis** -- identify new, modified, and removed stories/specs.
3. **Present change report** -- ask user: incremental update or full redesign.
4. **Apply changes** -- preserve stable sections, update affected ones, remove orphaned ones, re-map Story Resolution. Bump version (MAJOR if removed, MINOR if added/modified, PATCH if metadata). Set `:status: Current`.

### Post-implementation reconciliation

Invoked by the orchestrator after `needs-implementation` reports divergences where the user chose "update design".

For each routed divergence:
1. Locate and update relevant design sections
2. Ensure Story Resolution remains consistent
3. Verify no orphaned references or contradictions
4. Bump version (PATCH for minor clarifications, MINOR for structural changes)
5. Keep `:status: Current`

## Quality Checklist

- Every user story addressed in Story Resolution
- Every spec ID mapped to a design element (skip if `:source-spec-version: n/a`)
- ADR decisions respected
- Architecture constraints satisfied
- No unresolved unknowns (or explicitly listed)
- Design is implementable
- No cross-feature design dependencies
- Source versions recorded correctly
- At least one Mermaid component interaction diagram included

## Reference

See `references/example.adoc` for a complete example.
