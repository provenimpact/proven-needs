---
name: needs-spec
description: Create and maintain technical specifications with EARS requirements derived from user stories. Use when asked to create specifications, generate requirements from stories, update specs, sync specifications with user stories, or review specification coverage. Reads user-stories.adoc and produces categorized, black-box testable requirement files in docs/specs/.
---

## Prerequisites

Load the `proven-needs` skill first. It provides the overall workflow context, artifact locations, and conventions.

Load the `ears-requirements` skill before writing requirements. It provides the EARS sentence types and templates.

## Workflow

Creating and maintaining specifications involves these steps:

1. Read user stories
2. Check for existing specifications
3. Analyze and propose categories (ask user)
4. Derive black-box testable requirements
5. Write specification files
6. Sync workflow (when specs already exist)
7. Quality checklist

### 1. Read User Stories

Read `user-stories.adoc` from the project root. Extract:
- The `:version:` attribute
- All stories with their acceptance criteria
- Feature groupings

**If the file does not exist:** Stop and inform the user that user stories must be created first using the `needs-user-story` skill.

**If `:version:` is missing:** Inform the user that their `user-stories.adoc` needs version metadata. Instruct them to run the `needs-user-story` skill to add it, or ask if they want to proceed treating the current content as version `1.0.0`.

### 2. Check for Existing Specifications

Look for `docs/specs/` directory and `docs/specs/index.adoc`.

**If specs exist:** Follow the "Sync Workflow" (step 6).
**If no specs exist:** Continue with steps 3-5 to create initial specifications.

### 3. Analyze and Propose Categories

Analyze all user stories and acceptance criteria to identify natural requirement categories. Categories group requirements by technical domain, independent of which story they originate from.

Present the proposed categories to the user with prefixes and counts:

```
Proposed categories:
  1. Authentication (AUTH) - 8 requirements
  2. User Interface (UI) - 12 requirements
  3. Data Validation (DVAL) - 5 requirements
  4. Error Handling (ERR) - 6 requirements

Add, remove, rename, or merge categories?
```

**Prefix rules:**
- Each category gets a unique prefix of 2-5 uppercase characters
- Prefixes must be unique across all categories
- Prefixes are used in requirement IDs (e.g., `AUTH-001`)

Wait for user confirmation before proceeding.

### 4. Derive Requirements

For each acceptance criterion across all stories:
1. Determine which category it belongs to
2. Translate it into one or more black-box testable EARS requirements
3. Assign a unique ID: `<PREFIX>-<NNN>` (e.g., `AUTH-001`, `UI-003`)

**Black-box testing constraint -- the litmus test:**

> Could a tester who has never seen the source code verify this requirement using only the system's user interface or public APIs? If not, rewrite it.

Requirements must NOT reference:
- Internal architecture, components, or modules
- Database schemas, tables, or queries
- API endpoint paths or HTTP methods
- Programming languages, frameworks, or libraries
- Internal data structures or algorithms
- File system paths or internal configuration

Requirements MUST describe:
- What the user or external actor observes
- What inputs produce what outputs
- Observable system states and transitions
- Timing and performance from the user's perspective
- Error messages and feedback presented to the user

**Multi-source requirements:** A single requirement may be derived from criteria across multiple stories. Use comma-separated sources: `Source:: Story 1: Criterion 3, Story 2: Criterion 1`.

### 5. Write Specification Files

Create `docs/specs/` with one file per category plus an index.

**Index file (`docs/specs/index.adoc`):**

```asciidoc
= Specifications
:version: 1.0.0
:source-stories-version: <user-stories version>
:last-updated: YYYY-MM-DD
:toc:

== Categories

* <<authentication.adoc#,Authentication (AUTH)>> -- N requirements
* <<ui.adoc#,User Interface (UI)>> -- N requirements

== Traceability Matrix

[cols="1,2,2", options="header"]
|===
| Requirement ID | Requirement Summary | Source

| AUTH-001
| Registration form with email and password
| Story 1: User Registration, Criterion 1

| AUTH-002
| Email verification on registration
| Story 1: Criterion 3, Story 3: Criterion 1
|===
```

**Category file (e.g., `docs/specs/authentication.adoc`):**

```asciidoc
= Authentication Requirements
:version: 1.0.0
:last-updated: YYYY-MM-DD
:toc:

== AUTH-001
The system shall display a registration form with email and password fields.

Type:: Ubiquitous
Source:: Story 1: User Registration, Criterion 1
Verification:: Open the registration page. Confirm that email and password input fields are displayed.

== AUTH-002
When the user submits the registration form with valid data, the system shall send an email verification link.

Type:: Event-driven
Source:: Story 1: User Registration, Criterion 3
Verification:: Submit valid registration data and confirm that a verification email is received at the provided address.
```

Each requirement includes:
- The EARS requirement text as the section content
- `Type` -- the EARS sentence type used
- `Source` -- traceability to user story and criterion (comma-separated for multi-source)
- `Verification` -- a brief black-box test description

**Version rules for spec files:**
- `:version:` uses SemVer, starts at `1.0.0`
- `:source-stories-version:` (index only) records which user stories version was used
- Updated on every sync according to the bump rules in step 6

### 6. Sync Workflow

When specifications already exist:

#### 6.1 Quick Staleness Check

1. Read `:source-stories-version:` from `docs/specs/index.adoc`
2. Read `:version:` from `user-stories.adoc`
3. If versions match, inform the user that specs appear up to date. Ask if they want to force a re-analysis. If not, stop.

#### 6.2 Content-Based Change Analysis

The version check only detects staleness. Always perform a full content diff to determine actual changes:

1. Read all current user stories and their acceptance criteria
2. Read all existing requirements and their `Source` traceability links
3. For each story criterion, determine:
   - **New** -- no corresponding requirement exists -> derive new requirements
   - **Modified** -- the source criterion wording or intent changed -> update affected requirements
   - **Unchanged** -- criterion and corresponding requirement still align -> no action
4. For each existing requirement, check if its source criterion still exists:
   - **Orphaned** -- source criterion was removed -> mark for removal

#### 6.3 Category Analysis

- Check if new requirements fit existing categories or need new ones
- Check if any category has zero remaining requirements after removals
- **Only ask the user about categories when changes are needed** (new category proposed, or empty category to remove). Preserve existing categories by default.

#### 6.4 Present Change Report

```
Spec sync: user-stories 1.0.0 -> 1.1.0

Added:
  - UI-004: [new requirement summary]
  - UI-005: [new requirement summary]

Modified:
  - AUTH-002: [old text] -> [new text]

Removed:
  - AUTH-005: [reason - source Story 3 removed]

Categories:
  - Propose adding "Notifications" (NOTIF) -- 3 new requirements
  - Propose removing "Legacy" -- 0 remaining requirements

No changes to: ERR, DVAL, SEC
```

Ask the user to confirm before applying changes.

#### 6.5 Apply Changes and Bump Versions

After user confirmation:

| Change Type | Spec Version Bump |
|---|---|
| Requirements removed | MAJOR |
| Requirements added or modified | MINOR |
| Traceability or metadata updates only | PATCH |

- Update `:source-stories-version:` in `index.adoc` to match current user stories version
- Update `:version:` in all affected category files and `index.adoc`
- Update `:last-updated:` to today's date in all changed files
- Update the traceability matrix
- If a category has zero remaining requirements, remove the category file and its entry from the index

### 7. Quality Checklist

Before finalizing, verify every item:
- Every requirement uses the correct EARS sentence type
- Every requirement passes the black-box litmus test
- Every requirement has a unique ID with a valid 2-5 char prefix
- Every requirement traces back to at least one user story criterion
- Multi-source requirements list all sources
- No duplicate or conflicting requirements across categories
- No empty category files remain
- The traceability matrix is complete and matches all requirement files
- All category files have consistent format
- Version numbers are correct and updated
- `:source-stories-version:` matches the user stories `:version:`

## Reference

See `references/example.adoc` for a complete example showing how user stories are transformed into categorized specifications (initial creation) and how specifications are updated when user stories change (sync).
