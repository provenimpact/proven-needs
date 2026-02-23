---
name: needs-user-story
description: Generate user stories with EARS acceptance criteria from feature descriptions. Use when asked to create user stories, write stories for a feature, break down requirements into stories, or generate acceptance criteria for features.
---

## Prerequisites

Load the `proven-needs` skill first. It provides the overall workflow context, artifact locations, and conventions.

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

=== US-NNN: [Title]
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
- If file exists: read current content. Determine whether the user is adding new stories, modifying existing stories, or removing stories, then follow the appropriate workflow below.
- Preserve existing content and TOC
- Always write the result to the file

#### Adding New Stories

1. Check if the feature section already exists. If so, append under it. If not, create a new feature section.
2. Before appending, check for stories with substantially similar scope under the same feature. If a potential duplicate is found, present both to the user and ask whether to merge, replace, or keep both.
3. Assign the next sequential `US-NNN` ID after the highest existing ID in the file (zero-padded to 3 digits).
4. Bump the version (MINOR -- new content added).

#### Modifying Existing Stories

When the user asks to change, update, or rework existing stories:

1. Read the current stories and identify which stories the user wants to modify.
2. Present the proposed changes: show the current text alongside the new text for each story being modified.
3. Ask the user to confirm the changes before applying.
4. Bump the version based on the nature of the change:
   - Acceptance criteria fundamentally rewritten: MAJOR
   - Criteria refined or adjusted (non-breaking): MINOR
   - Typos, formatting, clarifications: PATCH

#### Removing Stories

When the user asks to remove stories:

1. Identify the stories to remove.
2. **Warn about downstream impact:** Removing stories may make specifications (`docs/specs/`), design (`docs/design/`), and tasks (`docs/tasks/`) stale. Inform the user that downstream artifacts should be re-synced after removal.
3. Ask the user to confirm the removal.
4. Remove the stories. Do not renumber remaining stories (IDs must remain stable for traceability).
5. If a feature section has no remaining stories, remove the section.
6. Bump the version: MAJOR (content removed).

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
