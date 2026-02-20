# proven-spec

Spec driven development workflow for creating production grade software.

Acceptance criteria and specifications follow the [EARS (Easy Approach to Requirements Syntax)](.opencode/skills/ears-requirements/reference/ears-reference.adoc) standard to ensure requirements are unambiguous, verifiable, and consistent.

## Workflow

```
Feature Description -> User Stories -> Specifications
                       (user-story)    (spec-manager)
```

1. Describe the feature you want to build
2. Generate user stories with acceptance criteria (`user-story` skill)
3. Derive categorized, black-box testable specifications (`spec-manager` skill)

## Skills

This project provides three [OpenCode skills](https://opencode.ai/docs/skills/) that work together:

### `user-story`

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

Subsequent runs append new stories to the same file under appropriate feature sections and bump the version.

### `spec-manager`

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

### `ears-requirements`

Write and review requirements using the EARS methodology. The agent loads this skill when writing, translating, or reviewing requirements in EARS format. Used as a prerequisite by both `user-story` and `spec-manager`.

## EARS Acceptance Criteria

Each acceptance criterion uses one of the EARS sentence types:

| Type | Pattern | Use for |
|------|---------|---------|
| Ubiquitous | The \<system\> shall \<response\>. | Always-on behavior |
| Event-driven | When \<trigger\>, the \<system\> shall \<response\>. | User actions or events |
| State-driven | While \<state\>, the \<system\> shall \<response\>. | Behavior during a state |
| Unwanted behavior | If \<trigger\>, then the \<system\> shall \<response\>. | Errors and edge cases |
| Optional | Where \<feature\>, the \<system\> shall \<response\>. | Feature-dependent behavior |

See the [EARS quick reference](.opencode/skills/ears-requirements/reference/ears-reference.adoc) for the full guide.

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

Given the user stories above, `spec-manager` produces categorized specifications:

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

- [EARS Quick Reference](.opencode/skills/ears-requirements/reference/ears-reference.adoc) -- Full guide to the EARS requirement syntax standard
