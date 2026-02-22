---
name: needs-architecture
description: Create and maintain the system architecture document. Use when asked to document the architecture, update the architecture, describe the system structure, generate an architecture overview, or when a design has been implemented and the architecture needs updating. Produces docs/architecture.adoc as a living document reflecting the current system state. Supports both greenfield projects (generated from design document) and existing projects (analyzed from codebase).
---

## Prerequisites

Load the `proven-needs` skill first. It provides the overall workflow context, artifact locations, and conventions.

## Workflow

Creating and maintaining the architecture document involves these steps:

1. Determine mode
2. Gather inputs
3. Generate or update the architecture document
4. Quality checklist

### 1. Determine Mode

Check for `docs/architecture.adoc` and the presence of a codebase (source files beyond documentation and configuration).

| Existing architecture doc? | Codebase exists? | Mode |
|---|---|---|
| No | No | **Greenfield** -- generate from design document |
| No | Yes | **Reverse-engineer** -- analyze codebase, use design doc if available |
| Yes | Any | **Update** -- refresh the existing architecture document |

### 2. Gather Inputs

Read these sources based on the mode:

**All modes:**
- `docs/adrs/` -- read all ADRs with status `Accepted` for technology decisions

**Greenfield mode:**
- `docs/design/design.adoc` -- **required**. The architecture is derived from the design. **If missing:** Stop and inform the user that a design document must be created first using the `needs-design` skill.

**Reverse-engineer mode:**
- Analyze the codebase: directory structure, key source files, configuration files (package.json, Cargo.toml, go.mod, etc.), entry points, module boundaries
- `docs/design/design.adoc` -- if available, use it to understand intended architecture alongside actual implementation

**Update mode:**
- Read existing `docs/architecture.adoc` and its `:version:` and `:source-design-version:`
- `docs/design/design.adoc` -- if available, compare its `:version:` against the architecture's `:source-design-version:`. If they differ, inform the user the architecture may be out of date relative to the design. If `:status:` is `Implemented`, incorporate the completed design changes.
- If codebase exists, verify architecture document matches actual implementation

### 3. Generate or Update the Architecture Document

Write `docs/architecture.adoc` using this format:

```asciidoc
= System Architecture
:version: 1.0.0
:source-design-version: <design version>
:last-updated: YYYY-MM-DD
:toc:

== Overview

<High-level description of the system, its purpose, and boundaries.>

== Technology Stack

<Languages, frameworks, databases, infrastructure.
 Reference relevant ADRs for the rationale behind each choice.>

* <<adrs/0001-use-typescript.adoc#,ADR-0001>>: TypeScript (backend and frontend)
* <<adrs/0002-use-postgresql.adoc#,ADR-0002>>: PostgreSQL (persistence)

== System Components

<Major components, modules, or services.
 For each: responsibility, key interfaces, dependencies.
 Adapt the structure to the project type.>

== Data Architecture

<Data stores, schemas, data flow between components.
 Reference docs/design/data-model.adoc if it exists.>

== External Interfaces

<APIs, integrations, user-facing interfaces.
 Reference docs/design/contracts/ if they exist.>

== Deployment Architecture

<How the system is deployed and operated.
 Skip if not applicable (e.g., a library).>

== Cross-Cutting Concerns

<Authentication, logging, error handling, monitoring, etc.
 Skip sections that do not apply.>
```

**Section guidance:**

The sections above are a default starting point. Adapt based on the project:

- **Libraries/SDKs:** Replace "Deployment Architecture" with "Distribution" (packaging, publishing). Replace "System Components" with "Module Structure".
- **CLI tools:** "External Interfaces" becomes "Command Interface". "Deployment" becomes "Installation".
- **Microservices:** Add "Service Communication" and "Service Discovery" sections.
- **Simple projects:** Merge or remove sections that add no value. A small CLI tool does not need separate "Data Architecture" and "Cross-Cutting Concerns" sections.

Remove sections that do not apply rather than leaving them empty.

**Version rules:**
- `:version:` uses SemVer, starts at `1.0.0`
- `:source-design-version:` records which design version the architecture was generated or updated from. Omit in reverse-engineer mode if no design document exists.
- MAJOR bump: components removed or fundamentally restructured
- MINOR bump: components added or modified
- PATCH bump: clarifications, formatting, metadata updates
- Always update `:last-updated:` to today's date

### 4. Quality Checklist

Before finalizing, verify:
- All ADR technology decisions are reflected in the Technology Stack section
- Component descriptions match the actual codebase (if one exists)
- No empty sections remain (remove instead)
- External interfaces match what the system actually exposes
- Data architecture covers all persistent data stores
- Version and date are updated

## Reference

See `references/example.adoc` for a complete example showing a greenfield architecture document for an e-commerce system.
