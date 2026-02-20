---
name: needs-user-story
description: Generate user stories with EARS acceptance criteria from feature descriptions. Use when asked to create user stories, write stories for a feature, break down requirements into stories, or generate acceptance criteria for features.
---

## Prerequisites

Load the `ears-requirements` skill before writing acceptance criteria. It provides the EARS sentence types and templates needed for criteria authoring.

## Workflow

### 1. Analyze the Input

Read the feature description. Identify:
- The main functionality requested
- Who the users are (roles)
- What problems they want solved
- Any specific requirements mentioned

### 2. Decompose into Stories

Break functionality into atomic user stories. Each story should:
- Be implementable in 1-3 days
- Deliver clear user value
- Have testable acceptance criteria

Common decomposition patterns:

| Feature Type | Typical Stories |
|---|---|
| Authentication | Login, Logout, Registration, Password Reset, Session Management |
| CRUD Operations | Create, Read, Update, Delete, List/Search |
| User Settings | View Settings, Update Settings, Preferences |
| Notifications | Subscribe, Receive, View History, Manage Preferences |

### 3. Write Each Story

For each story provide:

**Title:** Concise, descriptive (e.g. "User Registration")

**User story statement:**

```
As a [specific user role],
I want [specific action/feature],
so that [benefit/value].
```

**Acceptance criteria:** Write each criterion using the appropriate EARS sentence type from the `ears-requirements` skill. Each criterion must be specific, unambiguous, and verifiable. Cover happy path, edge cases, and error scenarios.

### 4. Structure the Output

Organize stories under feature headings using this AsciiDoc template:

```asciidoc
= User Stories
:version: 1.0.0
:last-updated: YYYY-MM-DD
:toc:

== [Feature Name]

=== Story N: [Title]
As a [role],
I want [goal],
so that [benefit].

Acceptance Criteria:
* [ ] The system shall [ubiquitous requirement].
* [ ] When [trigger], the system shall [response].
* [ ] If [error condition], then the system shall [response].
```

### 5. File Handling

- Target file: `user-stories.adoc` in project root
- If file does not exist: create with `= User Stories` header, `:version: 1.0.0`, `:last-updated:` set to today's date, and `:toc:` directive
- If file exists: read current content, append new stories under appropriate feature section or create new feature section
- Preserve existing content and TOC
- Always write the result to the file

### 6. Version Management

The `user-stories.adoc` file uses semantic versioning (SemVer).

**When creating the file for the first time:**
- Set `:version:` to `1.0.0`
- Set `:last-updated:` to today's date

**On every subsequent change, update both attributes:**

| Change Type | Bump | Example |
|---|---|---|
| Stories removed, acceptance criteria fundamentally rewritten | MAJOR | 1.0.0 -> 2.0.0 |
| New stories added, criteria modified (non-breaking) | MINOR | 1.0.0 -> 1.1.0 |
| Typos, formatting, clarifications (no behavioral change) | PATCH | 1.0.0 -> 1.0.1 |

- When bumping MAJOR, reset MINOR and PATCH to 0
- When bumping MINOR, reset PATCH to 0
- Always update `:last-updated:` to today's date

## Quality Checklist (INVEST)

Before outputting, verify:
- Each story has clear user value
- Acceptance criteria use the correct EARS sentence type
- Acceptance criteria are specific, unambiguous, and verifiable
- Stories are independent and can be implemented in any order
- No story is too large (break down if needed)
- Feature groupings make sense
- Error and edge case scenarios are covered using the "If ... then" (unwanted behavior) EARS type

INVEST criteria: **I**ndependent, **N**egotiable, **V**aluable, **E**stimable, **S**mall, **T**estable.

## Reference

See `references/example.adoc` for a complete example transformation showing how a feature description becomes structured user stories.
