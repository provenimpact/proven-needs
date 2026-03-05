---
name: needs-architecture
description: Create and maintain the system architecture document. Use when the proven-needs orchestrator determines the architecture document needs creating or updating. Operates at the project level, producing docs/architecture.adoc using the C4 model. Supports greenfield, reverse-engineering, and update modes.
---

## Prerequisites

Invoked by the `proven-needs` orchestrator with current state context.

## Observe

1. If `docs/architecture.adoc` exists: read `:version:`, `:last-updated:`, Feature Design Sources table, full content.
2. List all feature packages, check for `design.adoc` with `:version:` and `:status:`.
3. Read `docs/adrs/` (accepted) and `docs/constraints.adoc` (architecture constraints).
4. Analyze codebase: directory structure, config files, entry points, patterns.

## Evaluate

| Existing architecture? | Codebase exists? | Mode |
|---|---|---|
| No | No | **Greenfield** -- generate from feature designs |
| No | Yes | **Reverse-engineer** -- analyze codebase + designs |
| Yes | Any | **Update** -- refresh existing document |

**Greenfield:** at least one feature design must exist. **Update:** compare feature design versions against Feature Design Sources table to identify changes.

## Execute

### Generate or update the architecture document

Write `docs/architecture.adoc` using C4 model diagrams via Mermaid:

```asciidoc
= System Architecture
:version: 1.0.0
:last-updated: YYYY-MM-DD
:toc:

== Feature Design Sources
[cols="1,1", options="header"]
|===
| Feature | Design Version
| product-browsing | 1.0.0
|===

== Overview
<System purpose and boundaries.>

== System Context (C4 Level 1)
<Users and external systems.>
[source,mermaid]
----
C4Context
  ...
----

== Container View (C4 Level 2)
<Runtime containers and communication.>
[source,mermaid]
----
C4Container
  ...
----

== Component View (C4 Level 3)
<Internal structure of complex containers. Include only when meaningful.>

== Deployment View (C4 Level 4)
<Infrastructure topology. Include only for non-trivial deployments.>

== Technology Stack
<Languages, frameworks, DBs. Reference ADRs for rationale.>

== Data Architecture
<Data stores, schemas, flow.>

== Cross-Cutting Concerns
<Auth, logging, errors, monitoring. Skip sections that don't apply.>
```

### C4 level inclusion

Adaptive based on project complexity:

| Project Type | Levels |
|---|---|
| Libraries, SDKs, CLI tools | L1 + L2 only |
| Web apps (monolith or frontend+API) | L1 + L2 + L3 (backend/API) |
| Microservices / distributed | L1 + L2 + L3 + L4 |
| Mobile apps | L1 + L2 + L3 (app layers) |

Remove non-applicable sections rather than leaving placeholders.

### Section guidance

Adapt to the project: libraries may replace "Data Architecture" with "Data Flow", CLIs may add "Exit Codes" to Cross-Cutting, microservices may add "Service Discovery". Remove sections that add no value. Merge or simplify for simple projects.

**Feature integration:** Synthesize multiple feature designs into a cohesive system view without duplicating feature-specific details.

**Version rules:** MAJOR (components removed/restructured), MINOR (added/modified), PATCH (clarifications). Update Feature Design Sources table and `:last-updated:`.

## Quality Checklist

- C4 diagrams accurately reflect system structure
- All ADR decisions reflected in Technology Stack
- Component descriptions match codebase (if one exists)
- No empty sections (remove instead)
- Architecture constraints from `docs/constraints.adoc` addressed
- Feature Design Sources table current
- Version and date updated

## Reference

See `references/example.adoc` for a complete example.
