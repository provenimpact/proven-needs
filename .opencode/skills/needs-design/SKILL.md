---
name: needs-design
description: Create implementation design documents from user stories and specifications. Use when asked to design a feature, create an implementation plan, design the system, plan the implementation, or bridge specifications to code. Reads user-stories.adoc, docs/specs/, and docs/adrs/ to produce design artifacts in docs/design/. The design document solves the user stories, constrained by the specs (testable requirements) and ADRs (technology decisions).
---

## Prerequisites

Load the `proven-needs` skill first. It provides the overall workflow context, artifact locations, and conventions.

The `needs-adr` skill must be available. This skill invokes the ADR pattern when technology decisions are identified during the design process.

## Workflow

Creating an implementation design involves these steps:

1. Read inputs
2. Check for existing design
3. Phase 0: Research and decisions
4. Phase 1: Design
5. Write design files
6. Quality checklist

### 1. Read Inputs

Read these sources in order:

1. **`user-stories.adoc`** -- primary driver. Extract `:version:`, all stories, acceptance criteria, and feature groupings. **If missing:** Stop and inform the user that user stories must be created first using the `needs-user-story` skill.

2. **`docs/specs/index.adoc`** -- testable requirements. Extract `:version:`, all spec IDs and their categories. **If missing:** Stop and inform the user that specifications must be created first using the `needs-spec` skill.

3. **`docs/adrs/`** -- technology constraints and decisions already made. Read all ADRs with status `Accepted`. **If missing:** Note that no decisions exist yet; Phase 0 will address this.

4. **Existing codebase** -- if this is not a greenfield project, analyze the current code structure to understand what already exists. Look at directory structure, key source files, configuration files, and existing patterns.

### 2. Check for Existing Design

Look for `docs/design/design.adoc`.

**If no design exists:** Continue with Phase 0.

**If design exists:** Read `:status:`, `:source-stories-version:`, and `:source-specs-version:`.

| Condition | Action |
|---|---|
| `:status:` is `Implemented` | Previous design cycle is complete. Start a fresh design (overwrite). |
| Source versions match current stories and specs | Inform user design appears current. Ask to force re-design or stop. |
| Source versions differ | Design is stale. Present summary of what changed upstream. Ask user whether to redesign from scratch or incrementally update. |

### 3. Phase 0: Research and Decisions

Analyze all user stories and specs to identify:

1. **Technology decisions needed** -- for each story, what technology choices are required to solve it? Cross-reference against existing ADRs. Examples:
   - "Story 3 requires persistent storage -- which database?" (if no ADR exists)
   - "Story 5 requires real-time updates -- WebSocket or SSE?" (if no ADR exists)

2. **Unknowns and dependencies** -- external systems, third-party services, constraints that need clarification.

3. **Existing system analysis** (non-greenfield only) -- how is the current system structured? What patterns does it use? What can be reused vs. what needs modification?

**For each technology decision not covered by an existing ADR:**
- Present the decision to the user with context and alternatives
- Ask whether to create an ADR for it (use the `needs-adr` skill pattern)
- Record the decision in the design document regardless

**For each unknown:**
- Present it to the user as an open question
- Resolve before proceeding to Phase 1, or explicitly list as an open question in the design

### 4. Phase 1: Design

Design the solution that solves the user stories while satisfying the specs.

**The design structure is adaptive.** Choose an organization appropriate for the project type. The sections below are guidance, not a rigid template -- adapt, merge, split, or reorder sections based on what the project needs.

#### System design

Describe the major components, modules, or services that make up the solution. For each:
- Its responsibility (what it does)
- Its interfaces (how other components interact with it)
- Key implementation details

The structure depends on the project:

| Project Type | Typical Structure |
|---|---|
| Web application | Frontend components, backend services, database, API layer |
| CLI tool | Command parser, core logic modules, output formatters |
| Library/SDK | Public API surface, internal modules, extension points |
| Microservices | Service boundaries, communication patterns, shared infrastructure |
| Mobile app | Screens/views, state management, data layer, platform services |

#### Data model

If the system has persistent or structured data, describe:
- Entities and their attributes
- Relationships between entities
- Validation rules derived from specs
- State transitions (if applicable)

Write this to a separate file `docs/design/data-model.adoc` when the data model is non-trivial.

#### Interface contracts

If the system exposes external interfaces, describe the contracts:
- What interfaces exist (APIs, CLI commands, library exports, UI contracts)
- Input/output formats
- Error responses

Write these to `docs/design/contracts/` when relevant. Skip for purely internal projects.

#### Story resolution

For each user story (or logical group of related stories), describe:
- **Which components are involved** in solving this story
- **How the acceptance criteria are met** by the design
- **Which spec IDs are satisfied** by which design elements

This section is the proof that the design solves the stories. Every user story must appear here. Every spec ID must be mapped to at least one design element.

### 5. Write Design Files

Create `docs/design/` with the following files:

**Main design file (`docs/design/design.adoc`):**

```asciidoc
= Design Document
:version: 1.0.0
:status: Current
:source-stories-version: <user-stories version>
:source-specs-version: <specs version>
:last-updated: YYYY-MM-DD
:toc:

== Technical Context

<Project overview. Greenfield or existing system. Current state.>

== Decisions and Constraints

<References to relevant ADRs. Summary of technology decisions made during Phase 0.>

* <<../adrs/0001-use-typescript.adoc#,ADR-0001>>: Use TypeScript
* <<../adrs/0002-use-postgresql.adoc#,ADR-0002>>: Use PostgreSQL

== Research and Unknowns

<Findings from Phase 0. Resolved unknowns. Any remaining open questions.>

== System Design

<Adaptive structure -- components, modules, services, data flow.
 Organized appropriately for the project type.>

== Story Resolution

=== Story 1: <Title>

Components:: <which components are involved>
Criteria::
* <acceptance criterion> -> <how the design meets it> (SPEC-ID)
* ...

=== Story 2: <Title>

Components:: <which components are involved>
Criteria::
* ...
```

**`:status:` values:**
- `Current` -- design is valid and aligned with stories and specs
- `Stale` -- upstream stories or specs have changed since this design was created
- `Implemented` -- design has been fully implemented (set by a future implementation step)

**Version rules:**
- `:version:` uses SemVer, starts at `1.0.0`
- `:source-stories-version:` records which user stories version was used
- `:source-specs-version:` records which specs version was used
- `:last-updated:` set to today's date

**Additional files (when applicable):**
- `docs/design/data-model.adoc` -- entity/data model
- `docs/design/contracts/` -- interface contract files

### 6. Quality Checklist

Before finalizing, verify every item:
- Every user story is addressed in the Story Resolution section
- Every spec ID is mapped to at least one design element
- All ADR decisions are respected in the design
- No unresolved unknowns remain (or are explicitly listed as open questions)
- Design is implementable (specific enough to code from)
- Source versions are recorded correctly
- Data model covers all entities implied by the stories (if applicable)
- Interface contracts match the spec requirements (if applicable)

## Reference

See `references/example.adoc` for a complete example showing how user stories and specs become a design document with story resolution mapping.
