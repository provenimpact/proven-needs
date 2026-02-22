# proven-needs

Spec driven development workflow for creating production grade software.

Acceptance criteria and specifications follow the [EARS (Easy Approach to Requirements Syntax)](skills/ears-requirements/reference/ears-reference.adoc) standard to ensure requirements are unambiguous, verifiable, and consistent.

## Workflow

```
Feature Description -> User Stories -> Specifications -> Design -> Tasks -> Implementation
                       (needs-user-story)  (needs-spec)     |      (needs-tasks) (needs-implementation)
                                                        +----+----+
                                                        |         |
                                                      ADRs    Architecture
                                                   (needs-adr) (needs-architecture)
```

1. Describe the feature you want to build
2. Generate user stories with acceptance criteria (`needs-user-story` skill)
3. Derive categorized, black-box testable specifications (`needs-spec` skill)
4. Design the implementation that solves the stories (`needs-design` skill)
   - Record technology decisions as ADRs (`needs-adr` skill)
   - Document the resulting architecture (`needs-architecture` skill)
5. Create a phased implementation task list (`needs-tasks` skill)
6. Implement the code phase by phase (`needs-implementation` skill)

### Fast-Track

Not every project needs every step. The pipeline supports skipping intermediate steps when full traceability is not required. User stories are always the minimum input. For example, you can go directly from stories to design to implementation, skipping specs and tasks entirely. Each skill warns about the traceability impact of missing upstream artifacts and asks before proceeding in reduced mode. See the `proven-needs` skill for the full table of supported fast-track paths.

### Artifact Lifecycle

| Artifact | Location | Lifecycle |
|---|---|---|
| User Stories | `user-stories.adoc` | Living document, versioned with SemVer |
| Specifications | `docs/specs/` | Living document, synced with user stories |
| Design Document | `docs/design/` | Ephemeral -- invalid after stories/specs change or implementation completes |
| ADRs | `docs/adrs/` | Permanent, append-only records |
| Architecture | `docs/architecture.adoc` | Living document reflecting current system state |
| Tasks | `docs/tasks/` | Ephemeral -- stale when design changes, status tracks completion |
| Implemented Code | project source | Living -- the actual codebase produced from the task list |

## Skills

This project provides eight [OpenCode skills](https://opencode.ai/docs/skills/) that work together, plus a shared orchestrator (`proven-needs`) loaded by all workflow skills:

### `needs-user-story`

Generate user stories from feature descriptions. The agent loads this skill when asked to create user stories, break down requirements, or generate acceptance criteria.

Ask the agent to generate stories by describing what you want:

```
Create user stories for a todo app where users can create, edit, delete tasks and mark them complete
```

```
Write stories for social login with Google and GitHub alongside email/password authentication
```

The skill will:
1. Analyse the description and decompose it into atomic user stories
2. Write acceptance criteria using EARS sentence types
3. Output stories in AsciiDoc format to `user-stories.adoc` with SemVer versioning

Subsequent runs append new stories to the same file under appropriate feature sections and bump the version. The skill also supports modifying and removing existing stories with appropriate version management.

### `needs-spec`

Create and maintain technical specifications from user stories. The agent loads this skill when asked to create specifications, sync specs with stories, or review requirement coverage.

Ask the agent to generate specifications after creating user stories:

```
Create specifications from the user stories
```

```
Sync specifications with updated user stories
```

The skill will:
1. Read user stories and their version from `user-stories.adoc`
2. Analyze and propose requirement categories (with user confirmation)
3. Derive black-box testable EARS requirements
4. Write categorized specification files to `docs/specs/`
5. On subsequent runs, detect changes and sync specifications

### `needs-design`

Create implementation design documents that solve the user stories. The agent loads this skill when asked to design a feature, create an implementation plan, or bridge specifications to code.

Ask the agent to create a design after specs exist:

```
Design the implementation for the user stories
```

```
Create an implementation plan for the e-commerce features
```

The skill will:
1. Read user stories (primary driver), specs (testable requirements), and ADRs (technology decisions)
2. Identify technology decisions and unknowns -- offer to create ADRs for significant decisions
3. Design the system: components, data model, interfaces
4. Map every user story to its solution in the design (story resolution)
5. Write design artifacts to `docs/design/`

The design document tracks `:source-stories-version:` and `:source-specs-version:` and has a `:status:` field (`Current`, `Stale`, `Implemented`).

### `needs-adr`

Create and manage Architecture Decision Records. The agent loads this skill when asked to record a technical decision, document a technology choice, or when the design skill identifies decisions that need recording.

```
Record a decision to use PostgreSQL for the database
```

```
Create an ADR for choosing Next.js as the framework
```

The skill will:
1. Determine the next ADR sequence number
2. Gather context, decision, consequences, and alternatives
3. Write the ADR to `docs/adrs/NNNN-title.adoc`
4. Update `docs/adrs/index.adoc`

ADRs are permanent, append-only records. Decisions are never deleted -- they are superseded or deprecated.

### `needs-architecture`

Create and maintain the system architecture document. The agent loads this skill when asked to document or update the architecture, or after a design has been implemented.

```
Document the system architecture
```

```
Update the architecture after implementing the new features
```

The skill will:
1. **Greenfield projects:** Generate architecture from the design document
2. **Existing projects:** Analyze the codebase and reconcile with the design document
3. Write `docs/architecture.adoc` as a living snapshot of the current system

### `needs-tasks`

Create phased implementation task lists from design documents. The agent loads this skill when asked to break down the design into work items, plan the implementation, or prepare for coding.

```
Create implementation tasks from the design
```

```
Break down the design into a task list
```

The skill will:
1. Read the design document, user stories, and specs
2. Decompose the design into discrete coding units
3. Organize tasks into sequential phases (Foundation, Core Logic, Interface Layer, Integration, Polish)
4. Mark each task as `[parallel]` or `[sequential]` within its phase
5. Write `docs/tasks/tasks.adoc` with full traceability back to specs and stories

### `needs-implementation`

Implement code from the phased task list. The agent loads this skill when asked to start coding, implement the tasks, or build the feature.

```
Implement the tasks
```

```
Start coding from the task list
```

The skill will:
1. Read the task list and design document
2. Build a todo list for the current phase
3. Implement each task following the design document for architectural guidance
4. Verify the phase (build, lint, typecheck, test)
5. Commit with a descriptive message
6. Ask the user whether to continue to the next phase or stop
7. When all phases are complete, mark the design and tasks as `Implemented`

### `ears-requirements`

Write and review requirements using the EARS methodology. The agent loads this skill when writing, translating, or reviewing requirements in EARS format. Used as a prerequisite by both `needs-user-story` and `needs-spec`.

## EARS Acceptance Criteria

Each acceptance criterion uses one of the EARS sentence types:

| Type | Pattern | Use for |
|------|---------|---------|
| Ubiquitous | The \<system\> shall \<response\>. | Always-on behavior |
| Event-driven | When \<trigger\>, the \<system\> shall \<response\>. | User actions or events |
| State-driven | While \<state\>, the \<system\> shall \<response\>. | Behavior during a state |
| Unwanted behavior | If \<trigger\>, then the \<system\> shall \<response\>. | Errors and edge cases |
| Optional | Where \<feature\>, the \<system\> shall \<response\>. | Feature-dependent behavior |

See the [EARS quick reference](skills/ears-requirements/reference/ears-reference.adoc) for the full guide.

## Example: User Stories

Given: "user login and registration system"

```asciidoc
= User Stories
:version: 1.0.0
:last-updated: 2026-02-20
:toc:

== Authentication

=== Story 1: User Registration
As a new user,
I want to create an account with email and password,
so that I can access the application.

Acceptance Criteria:
* [ ] The system shall display a registration form with email and password fields.
* [ ] The system shall enforce minimum password security requirements.
* [ ] When the user submits the registration form, the system shall send an email verification link.
* [ ] If the user submits an email that is already registered, then the system shall display an error message.
* [ ] When registration is successful, the system shall redirect to the login page or dashboard.

=== Story 2: User Login
As a registered user,
I want to log in with email and password,
so that I can access my account.

Acceptance Criteria:
* [ ] The system shall display a login form with email and password fields.
* [ ] When the user submits valid credentials, the system shall redirect to the dashboard.
* [ ] If the user submits invalid credentials, then the system shall display an error message.
* [ ] The system shall provide a "remember me" option for persistent sessions.
* [ ] The system shall provide a "forgot password" link on the login form.

=== Story 3: Password Reset
As a user who forgot their password,
I want to reset my password via email,
so that I can regain access to my account.

Acceptance Criteria:
* [ ] The system shall display a forgot-password form with an email field.
* [ ] When the user submits a registered email, the system shall send a password reset link.
* [ ] The system shall expire the reset link after a configured time period.
* [ ] The system shall enforce minimum password security requirements for the new password.
* [ ] When the password is successfully reset, the system shall display a confirmation message.
```

## Example: Specifications

Given the user stories above, `needs-spec` produces categorized specifications:

```asciidoc
= Authentication Requirements
:version: 1.0.0
:last-updated: 2026-02-20
:toc:

== AUTH-001
The system shall display a registration form with email and password fields.

Type:: Ubiquitous
Source:: Story 1: User Registration, Criterion 1
Verification:: Open the registration page. Confirm that email and password input fields are displayed.

== AUTH-002
The system shall enforce minimum password security requirements.

Type:: Ubiquitous
Source:: Story 1: User Registration, Criterion 2
Verification:: Attempt to register with passwords of varying strength. Confirm that weak passwords are rejected with a descriptive message.

== AUTH-003
When the user submits the registration form with valid data, the system shall send an email verification link.

Type:: Event-driven
Source:: Story 1: User Registration, Criterion 3
Verification:: Submit valid registration data and confirm that a verification email is received at the provided address.
```

## Reference

- [EARS Quick Reference](skills/ears-requirements/reference/ears-reference.adoc) -- Full guide to the EARS requirement syntax standard
