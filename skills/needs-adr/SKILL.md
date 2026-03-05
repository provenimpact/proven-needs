---
name: needs-adr
description: Create and manage Architecture Decision Records (ADRs). Use when the proven-needs orchestrator identifies technology decisions that need recording, or when explicitly asked to record a decision. Operates at the project level in docs/adrs/. ADRs are permanent, append-only records referenced by feature designs.
---

## Prerequisites

Invoked by the `proven-needs` orchestrator or by `needs-design` when technology decisions are identified during the design process.

## Observe

1. Check for `docs/adrs/` and `docs/adrs/index.adoc`. If exists, read all ADR numbers, titles, statuses, dates, and next available sequence number. If not, next number is `0001`.
2. Read `docs/constraints.adoc` for relevant constraints (architecture, licensing).

## Evaluate

1. Search existing ADRs for the same topic. Already covered → no action. Decision changed → supersession needed. Not covered → new ADR needed.
2. Check if proposed technology violates any constraint. Flag conflicts.

## Execute

### Creating a new ADR

#### 1. Gather decision context

Identify: Context (what issue?), Decision (what approach?), Consequences (what trade-offs?), Alternatives (what else considered?).

- When invoked standalone: ask the user
- When invoked from `needs-design`: design capability provides context, user confirms

#### 2. Write the ADR file

Create `docs/adrs/NNNN-<kebab-case-title>.adoc`:

```asciidoc
= ADR-NNNN: <Decision Title>
:status: Accepted
:date: YYYY-MM-DD
:deciders: <who was involved>

== Context
<What motivates this decision?>

== Decision
<What change is being made?>

== Consequences
<What becomes easier or harder?>

== Alternatives Considered

=== <Alternative 1>
<Description and reason for rejection>
```

**Status values:** `Proposed` (unconfirmed), `Accepted` (in effect), `Deprecated` (no longer relevant), `Superseded by ADR-NNNN` (replaced).

**Numbering:** 4-digit zero-padded (`0001`, `0002`, ...). File naming: `NNNN-kebab-case-title.adoc`.

#### 3. Update the index

Create or update `docs/adrs/index.adoc`:

```asciidoc
= Architecture Decision Records
:version: 1.0.0
:last-updated: YYYY-MM-DD
:toc:

[cols="1,3,1,1", options="header"]
|===
| ADR | Decision | Status | Date
| <<0001-use-typescript.adoc#,ADR-0001>> | Use TypeScript for backend | Accepted | 2026-02-20
|===
```

Index version: MINOR when ADRs added, PATCH for status updates.

### Superseding an existing ADR

1. Set old ADR status to `Superseded by ADR-NNNN`
2. Create new ADR referencing the old one in Context
3. Update index

### Deprecating an ADR

1. Set status to `Deprecated`
2. Update index

Never delete ADR files. ADRs are append-only.

## Reference

See `references/example.adoc` for a complete example.
