# proven-spec

Spec driven development workflow for creating production grade software.

Acceptance criteria follow the [EARS (Easy Approach to Requirements Syntax)](ears-reference.adoc) standard to ensure requirements are unambiguous, verifiable, and consistent.

## Usage

Generate user stories from feature descriptions using the `/story` command.

```
/story <description of what you want the software to do>
```

### Examples

#### Basic Usage

```
/story I want a todo app where users can create, edit, delete tasks and mark them complete
```

#### More Detailed Description

```
/story Build an e-commerce platform with product catalog, shopping cart, user authentication, and payment processing
```

#### Specific Feature

```
/story Add social login with Google and GitHub alongside email/password authentication
```

### Workflow

1. Run `/story` with your description
2. AI generates user stories in AsciiDoc format with EARS acceptance criteria
3. Review the generated stories in `user-stories.adoc`

### File Output

Stories are saved to `user-stories.adoc` with:
- Table of Contents as index
- Stories grouped by feature
- Each story includes acceptance criteria written using EARS sentence types

Subsequent runs of `/story` append new stories to the same file under appropriate feature sections.

## EARS Acceptance Criteria

Each acceptance criterion uses one of the EARS sentence types:

| Type | Pattern | Use for |
|------|---------|---------|
| Ubiquitous | The \<system\> shall \<response\>. | Always-on behavior |
| Event-driven | When \<trigger\>, the \<system\> shall \<response\>. | User actions or events |
| State-driven | While \<state\>, the \<system\> shall \<response\>. | Behavior during a state |
| Unwanted behavior | If \<trigger\>, then the \<system\> shall \<response\>. | Errors and edge cases |
| Optional | Where \<feature\>, the \<system\> shall \<response\>. | Feature-dependent behavior |

See [ears-reference.adoc](ears-reference.adoc) for the full EARS reference guide.

## Example Output

Given: `/story user login and registration system`

```asciidoc
= User Stories

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

## Reference

- [EARS Quick Reference](ears-reference.adoc) -- Full guide to the EARS requirement syntax standard
