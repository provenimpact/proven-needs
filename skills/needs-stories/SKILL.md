---
name: needs-stories
description: Create and update user stories with EARS acceptance criteria for a feature. Use when the proven-needs orchestrator determines that a feature needs user stories created or updated. Operates within a single feature package at docs/features/<slug>/. Stories explain WHY from the user's perspective — the needs, motivations, and value a feature delivers.
---

## Prerequisites

Load the `ears-requirements` skill before writing acceptance criteria.

Invoked by the `proven-needs` orchestrator with feature context (slug, intent, current state).

## Observe

1. Check if `docs/features/<slug>/` exists. If not, this is a new feature.
2. If `user-stories.adoc` exists: read `:version:`, `:last-updated:`, all story IDs, titles, and acceptance criteria.
3. Read `docs/constraints.adoc` for quality constraints (testability, completeness).

## Evaluate

1. No stories exist and intent requires them → create. Stories exist but intent adds new functionality → add. Intent modifies existing behavior → modify. Stories fully cover the desired state → no action.
2. Verify proposed stories: must be testable, must not duplicate constraint-level requirements, must be scoped to this one feature.

## Execute

### Creating stories for a new feature

#### 1. Analyze the intent

Identify: main functionality, user roles, problems to solve, specific requirements mentioned.

#### 2. Decompose into stories

Each story must: be implementable in 1-3 days, deliver clear user value, have testable acceptance criteria, be scoped entirely within this feature. If a story seems too broad, split it or flag it for feature decomposition.

Common patterns:

| Feature Type | Typical Stories |
|---|---|
| Authentication | Login, Logout, Registration, Password Reset, Session Management |
| CRUD Operations | Create, Read, Update, Delete, List/Search |
| User Settings | View Settings, Update Settings, Preferences |
| Notifications | Subscribe, Receive, View History, Manage Preferences |

#### 3. Write each story

**User story statement:**
```
As a [specific user role],
I want [specific action/feature],
so that [benefit/value].
```

**Acceptance criteria:** Use the appropriate EARS sentence type from `ears-requirements`. Each criterion must be specific, unambiguous, and verifiable. Cover happy path, edge cases, and error scenarios.

#### 4. Check for constraint-level requirements

For each criterion: if it applies only to this feature → keep as criterion. If it would apply to other features too → flag to the orchestrator as a potential constraint.

#### 5. Write the file

Create `docs/features/<slug>/user-stories.adoc`:

```asciidoc
= User Stories: <Feature Name>
:version: 1.0.0
:last-updated: YYYY-MM-DD
:feature: <slug>
:toc:

== US-001: <Title>
As a [role],
I want [goal],
so that [benefit].

Acceptance Criteria:

* [ ] The system shall [ubiquitous requirement].
* [ ] When [trigger], the system shall [response].
* [ ] If [error condition], then the system shall [response].

== US-002: <Title>
...
```

Story IDs are sequential within this feature (US-001, US-002, ...), unique within the feature only.

### Adding stories to an existing feature

1. Read existing stories and next available US-NNN ID.
2. Check for substantially similar scope. If duplicate found, present both and ask whether to merge, replace, or keep both.
3. Assign next sequential ID. MINOR version bump. Update `:last-updated:`.

### Modifying existing stories

1. Present proposed changes: current text alongside new text.
2. Ask user to confirm.
3. Version bump: MAJOR (fundamentally rewritten), MINOR (refined/adjusted), PATCH (typos/formatting).

### Removing stories

1. Warn about downstream impact (spec and design may become stale).
2. Ask user to confirm. Remove stories. Do NOT renumber remaining stories (IDs are stable).
3. MAJOR version bump.

## Quality Checklist (INVEST)

- Each story has clear user value
- Acceptance criteria use the correct EARS sentence type
- Criteria are specific, unambiguous, and verifiable
- Stories are independent and can be implemented in any order
- No story is too large
- Error and edge cases covered using "If ... then" EARS type
- No story spans multiple features
- Cross-cutting requirements flagged as potential constraints
- Version and date updated

INVEST: **I**ndependent, **N**egotiable, **V**aluable, **E**stimable, **S**mall, **T**estable.

## Reference

See `references/example.adoc` for a complete example.
