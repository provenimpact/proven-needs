---
name: needs-spec
description: Create and maintain feature specifications with EARS requirements derived from user stories. Use when the proven-needs orchestrator determines that a feature needs specifications created or synced. Operates within a single feature package at docs/features/<slug>/. Specs define WHAT must be true — black-box testable requirements for externally observable behavior.
---

## Prerequisites

Load the `ears-requirements` skill before writing requirements.

Invoked by the `proven-needs` orchestrator with feature context (slug, intent, current state).

## Observe

1. Read `docs/features/<slug>/user-stories.adoc`: extract `:version:` and all stories with acceptance criteria. If missing, report to orchestrator.
2. If `docs/features/<slug>/spec.adoc` exists: read `:version:`, `:source-stories-version:`, all requirement IDs, texts, types, sources, and verifications.
3. Read `docs/constraints.adoc` to identify constraints that overlap with this feature's domain (these should not be duplicated as specs).

## Evaluate

1. No spec exists → create. Spec exists but stories version differs → sync. Versions match → check content alignment, report if current.
2. Requirements duplicating a project-wide constraint should not appear in the spec. If a spec requirement has since been promoted to a constraint, flag it for removal.

## Execute

### Creating a new spec

#### 1. Derive requirements

For each acceptance criterion across all stories, translate into one or more black-box testable EARS requirements. Assign unique IDs: `<PREFIX>-<NNN>` where PREFIX is 2-5 uppercase characters from the feature slug (e.g., `product-browsing` → `PROD`, `shopping-cart` → `CART`). Present the proposed prefix to the user for confirmation.

**Black-box litmus test:**

> Could a tester who has never seen the source code verify this requirement using only the UI or public APIs? If not, rewrite it.

Requirements must NOT reference internal architecture, database schemas, API paths, languages/frameworks, internal data structures, or file system paths.

Requirements MUST describe what the user observes, what inputs produce what outputs, observable state transitions, timing from the user's perspective, and error feedback.

**Constraint filtering:** If a requirement is already covered by `docs/constraints.adoc`, don't include it. Add a NOTE instead:
```asciidoc
NOTE: Password strength requirements are enforced by project constraint (Security). Not duplicated here.
```

**Multi-source requirements:** Use comma-separated sources: `Source:: US-001: Criterion 3, US-002: Criterion 1`.

#### 2. Write the spec file

Create `docs/features/<slug>/spec.adoc`:

```asciidoc
= Specification: <Feature Name>
:version: 1.0.0
:source-stories-version: <user-stories version>
:last-updated: YYYY-MM-DD
:feature: <slug>
:prefix: <PREFIX>
:toc:

== <PREFIX>-001
The system shall display products in a grid or list format.

Type:: Ubiquitous
Source:: US-001: View Product Catalog, Criterion 1
Verification:: Open the product catalog page. Confirm products display in grid or list layout.

== <PREFIX>-002
When the user selects a category filter, the system shall display only products in that category.

Type:: Event-driven
Source:: US-001: View Product Catalog, Criterion 3
Verification:: Select a category filter. Confirm only matching products display.

== Traceability

[cols="1,2,2", options="header"]
|===
| Requirement ID | Requirement Summary | Source
| <PREFIX>-001 | Product display format | US-001: Criterion 1
| <PREFIX>-002 | Category filtering | US-001: Criterion 3
|===
```

### Syncing an existing spec

#### 1. Quick staleness check

Compare `:source-stories-version:` against stories `:version:`. If they match, report current to orchestrator.

#### 2. Content-based change analysis

For each story criterion determine: **New** (no requirement exists), **Modified** (wording/intent changed), **Unchanged**. For each existing requirement check: **Orphaned** (source removed), **Promoted to constraint** (now in `docs/constraints.adoc`).

#### 3. Present change report

```
Spec sync: stories 1.0.0 -> 1.1.0

Added:   PROD-005: [summary]
Modified: PROD-002: [old] -> [new]
Removed:  PROD-004: source US-003 removed
          PROD-006: promoted to project constraint
No changes: PROD-001, PROD-003
```

Ask user to confirm before applying.

#### 4. Apply changes and bump versions

| Change Type | Version Bump |
|---|---|
| Requirements removed | MAJOR |
| Requirements added or modified | MINOR |
| Traceability or metadata only | PATCH |

Update `:source-stories-version:`, `:version:`, `:last-updated:`, and traceability section.

## Quality Checklist

- Every requirement uses the correct EARS sentence type
- Every requirement passes the black-box litmus test
- Every requirement has a unique ID with the feature prefix
- Every requirement traces back to at least one story criterion
- No duplicates, conflicts, or constraint overlaps
- Traceability section is complete
- Version numbers correct and `:source-stories-version:` matches stories

## Reference

See `references/example.adoc` for a complete example.
