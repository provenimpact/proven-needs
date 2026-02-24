---
name: needs-architecture
description: Create and maintain the system architecture document. Use when the proven-intent orchestrator determines the architecture document needs creating or updating. Operates at the project level, producing docs/architecture.adoc as a living document reflecting the current system state. Supports greenfield projects (from feature designs), existing projects (from codebase analysis), and updates after feature implementations.
---

## Prerequisites

This skill is invoked by the `proven-intent` orchestrator, which provides the current state context.

## Observe

Assess the current state of architecture documentation.

### 1. Check for existing architecture document

If `docs/architecture.adoc` exists:
- Read `:version:`, `:source-design-version:`, `:last-updated:`
- Read the full content

### 2. Check for feature designs

List all feature packages in `docs/features/`. For each, check if `design.adoc` exists and read its `:version:` and `:status:`.

### 3. Read project-wide artifacts

- **`docs/adrs/`** -- read all ADRs with status `Accepted`
- **`constraints.adoc`** -- read architecture constraints

### 4. Analyze codebase

Check for source files beyond documentation and configuration. Understand:
- Directory structure, module organization
- Configuration files (package.json, Cargo.toml, go.mod, etc.)
- Entry points, key patterns

### 5. Report observation

Return to the orchestrator:
```
Architecture: {exists: true/false, version: "X.Y.Z", source-design-version: "X.Y.Z"}
Feature designs: [{slug: "...", version: "X.Y.Z", status: "..."}]
Codebase: {exists: true/false, language: "...", framework: "..."}
ADRs: {count: N, accepted: N}
Mode: greenfield / reverse-engineer / update
```

## Evaluate

Given the desired state from the orchestrator, determine what action is needed.

### 1. Determine mode

| Existing architecture? | Codebase exists? | Mode |
|---|---|---|
| No | No | **Greenfield** -- generate from feature designs |
| No | Yes | **Reverse-engineer** -- analyze codebase, use designs if available |
| Yes | Any | **Update** -- refresh the existing architecture document |

### 2. Check preconditions

**Greenfield mode:** At least one feature design must exist. If not, report to orchestrator that designs are needed first.

**Update mode:** Compare feature design versions against `:source-design-version:` in the architecture document. If designs have been updated or new features implemented since last architecture update, note what has changed.

### 3. Check constraints

Verify that the architecture description will address all architecture constraints from `constraints.adoc`.

### 4. Report evaluation

Return to the orchestrator:
```
Action: create / update / none
Mode: greenfield / reverse-engineer / update
Changes since last update: [list of feature designs that changed]
Constraint issues: [list or none]
```

## Execute

### Generate or update the architecture document

Write `docs/architecture.adoc`:

```asciidoc
= System Architecture
:version: 1.0.0
:source-design-version: <latest feature design version or "multiple">
:last-updated: YYYY-MM-DD
:toc:

== Overview

<High-level description of the system, its purpose, and boundaries.>

== Technology Stack

<Languages, frameworks, databases, infrastructure.
 Reference relevant ADRs for rationale.>

* <<adrs/0001-use-typescript.adoc#,ADR-0001>>: TypeScript (backend and frontend)
* <<adrs/0002-use-postgresql.adoc#,ADR-0002>>: PostgreSQL (persistence)

== System Components

<Major components, modules, or services.
 For each: responsibility, key interfaces, dependencies.
 Adapt structure to the project type.>

== Data Architecture

<Data stores, schemas, data flow between components.>

== External Interfaces

<APIs, integrations, user-facing interfaces.>

== Deployment Architecture

<How the system is deployed and operated.
 Skip if not applicable (e.g., a library).>

== Cross-Cutting Concerns

<Authentication, logging, error handling, monitoring, etc.
 Skip sections that do not apply.>
```

**Section guidance:**

The sections above are a default starting point. Adapt based on the project:

- **Libraries/SDKs:** Replace "Deployment Architecture" with "Distribution". Replace "System Components" with "Module Structure".
- **CLI tools:** "External Interfaces" becomes "Command Interface". "Deployment" becomes "Installation".
- **Microservices:** Add "Service Communication" and "Service Discovery" sections.
- **Simple projects:** Merge or remove sections that add no value.

Remove sections that do not apply rather than leaving them empty.

**Feature integration:** When multiple feature designs exist, the architecture document synthesizes them into a cohesive system view. It shows how features relate at the system level without duplicating feature-specific design details.

**Version rules:**
- `:version:` uses SemVer, starts at `1.0.0`
- `:source-design-version:` records which design versions the architecture reflects. Use "multiple" and list in the document body when synthesizing from multiple feature designs.
- MAJOR bump: components removed or fundamentally restructured
- MINOR bump: components added or modified
- PATCH bump: clarifications, formatting, metadata updates
- Always update `:last-updated:` to today's date

## Quality Checklist

Before finalizing, verify:
- All ADR technology decisions are reflected in Technology Stack
- Component descriptions match the actual codebase (if one exists)
- No empty sections remain (remove instead)
- External interfaces match what the system actually exposes
- Data architecture covers all persistent data stores
- Architecture constraints from `constraints.adoc` are addressed
- Version and date are updated

## Reference

See `references/example.adoc` for a complete example showing an architecture document for the e-commerce system.
