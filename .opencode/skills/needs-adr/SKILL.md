---
name: needs-adr
description: Create and manage Architecture Decision Records (ADRs) in docs/adrs/. Use when asked to record a technical decision, document a technology choice, create an ADR, list or review existing decisions, update or supersede a decision, or when the needs-design skill identifies decisions that need recording.
---

## Prerequisites

Load the `proven-needs` skill first. It provides the overall workflow context, artifact locations, and conventions.

## Workflow

Creating and managing ADRs involves these steps:

1. Read existing ADRs
2. Gather decision context
3. Write the ADR file
4. Update the index

### 1. Read Existing ADRs

Look for `docs/adrs/` and `docs/adrs/index.adoc`.

**If the directory exists:** Read `index.adoc` to understand existing decisions and determine the next sequence number.
**If no directory exists:** Create `docs/adrs/` and start numbering at `0001`.

### 2. Gather Decision Context

For each decision, identify:
- **Context** -- what is the issue motivating this decision?
- **Decision** -- what is the chosen approach?
- **Consequences** -- what becomes easier or harder?
- **Alternatives considered** -- what other options were evaluated and why were they rejected?

**When invoked standalone:** Ask the user for this information.
**When invoked from `needs-design`:** The design skill provides the context and asks the user to confirm the decision.

### 3. Write the ADR File

Create `docs/adrs/NNNN-<kebab-case-title>.adoc` using this format:

```asciidoc
= ADR-NNNN: <Decision Title>
:status: Accepted
:date: YYYY-MM-DD
:deciders: <who was involved>

== Context

<What is the issue motivating this decision?>

== Decision

<What is the change that is being made?>

== Consequences

<What becomes easier or harder because of this change?>

== Alternatives Considered

=== <Alternative 1>
<Description and reason for rejection>

=== <Alternative 2>
<Description and reason for rejection>
```

**Status values:**
- `Proposed` -- decision not yet confirmed
- `Accepted` -- decision is in effect
- `Deprecated` -- decision is no longer relevant
- `Superseded by ADR-NNNN` -- replaced by a newer decision

**Numbering:** Always use 4-digit zero-padded sequence numbers (`0001`, `0002`, ...).

**File naming:** `NNNN-kebab-case-title.adoc` (e.g., `0001-use-typescript.adoc`).

### 4. Update the Index

Create or update `docs/adrs/index.adoc`:

```asciidoc
= Architecture Decision Records
:toc:

[cols="1,3,1,1", options="header"]
|===
| ADR | Decision | Status | Date

| <<0001-use-typescript.adoc#,ADR-0001>>
| Use TypeScript for backend
| Accepted
| 2026-02-20

| <<0002-use-postgresql.adoc#,ADR-0002>>
| Use PostgreSQL for persistence
| Accepted
| 2026-02-20
|===
```

### Updating Existing ADRs

When a decision changes:
1. Set the old ADR status to `Superseded by ADR-NNNN`
2. Create a new ADR referencing the old one in its Context section
3. Update the index

When a decision becomes irrelevant:
1. Set the ADR status to `Deprecated`
2. Update the index

Never delete ADR files. ADRs are append-only records.

## Reference

See `references/example.adoc` for a complete example showing an ADR index and two ADR files.
