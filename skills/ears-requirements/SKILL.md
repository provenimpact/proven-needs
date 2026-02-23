---
name: ears-requirements
description: Write and review requirements using the EARS (Easy Approach to Requirements Syntax) methodology. Guides structured requirement authoring with correct sentence types, identifies missing requirements, and ensures quality characteristics.
---

## What I do

Guide the writing, review, and translation of requirements using the EARS (Easy Approach to Requirements Syntax) methodology.

Use the bundled quick reference for the canonical patterns, rules, and checklists: `reference/ears-reference.adoc`.

## General Rules

- Use: "The <system/component> shall <behavior>".
- Prefer one primary behavior per requirement; split "and/or" chains.
- Use observable, verifiable language (formats, limits, timings, state changes).
- Put conditions in the EARS prefix; keep the "shall" clause focused on the response.
- Name actors, states, events, and data consistently.

## EARS Sentence Types

Every requirement must use one of the following sentence types. Choose the type based on the nature of the requirement.

### Ubiquitous

Requirements that apply at all times without a specific trigger or state.

**Template:** The `<system name>` shall `<system response>`.

**Example:** The API shall return responses in JSON format.

### Event-driven

Requirements triggered by a specific event, optionally with preconditions.

**Template:** When `<optional preconditions>` `<trigger>`, the `<system>` shall `<system response>`.

**Examples:**

- When the user submits a valid order, the system shall create an order record.
- When a payment is declined, the system shall notify the user.

### State-driven

Requirements that apply while the system is in a particular state.

**Template:** While `<in a state>`, the `<system>` shall `<system response>`.

For readability, you may use "During" instead of "While" with the same meaning.

**Example:** While in Maintenance Mode, the service shall reject non-admin requests with HTTP 503.

### Unwanted behavior

Requirements that handle error conditions or undesirable inputs.

**Template:** If `<optional preconditions>` `<trigger>`, then the `<system>` shall `<system response>`.

**Examples:**

- If the uploaded file exceeds 10 MB, the system shall reject the upload and display an error message.
- If the database is unavailable, the service shall return HTTP 503 within 2 seconds.

### Optional feature

Requirements that only apply when a specific feature is present.

**Template:** Where `<feature is included>`, the `<system>` shall `<system response>`.

**Examples:**

- Where two-factor authentication is enabled, the system shall require a one-time code at login.
- Where audit logging is enabled, the service shall record an audit entry for each administrative change.

### Complex (combined)

Requirements that combine a precondition (state/condition/feature) with an event trigger.

**Templates:**

- While `<state>`, when `<trigger>`, the `<system>` shall `<system response>`.
- Where `<feature is included>`, while `<state>`, when `<trigger>`, the `<system>` shall `<system response>`.

You can also nest Where/While/When context inside If ... then statements for unwanted behaviour.

### Combined Sentences

Sentence types can be combined for complex requirements. Use the keywords in this order when combining: **Where** ... **While** ... **When** ... / **If** ... **then** ...

**Examples:**

- While the user is unauthenticated, when the user requests the account page, the system shall redirect to the login page.
- Where the cart is empty, when the user selects Checkout, the system shall display a message that checkout is unavailable.

## How to Apply EARS

Follow these steps when writing or translating requirements:

1. **Identify** whether you are working with a requirement, or something else (e.g. a note, example, or design decision). Only requirements get EARS treatment.
2. **Identify compound requirements** -- determine whether the requirement needs to be split into multiple atomic requirements.
3. **Identify the acting system, person, or process.**
4. **Analyse the needed sentence type(s)** -- select from Ubiquitous, Event-driven, State-driven, Unwanted behavior, or Optional feature.
   Use Complex (combined) when you need both preconditions (Where/While) and a trigger (When), including within If-then statements for unwanted behavior.
5. **Identify possible missing requirements** -- e.g. 2 states and 2 events usually produce 4 requirements. Check all combinations.
6. **Analyse the translated requirements** for ambiguity, conflict, and repetition.
7. **Review requirements** if possible. Iterate as required.

## Choosing a Pattern

- Always true -> Ubiquitous
- Triggered by an event -> Event-driven
- Depends on a state/mode -> State-driven
- Depends on a feature -> Optional feature
- Error/invalid/failure handling -> Unwanted behavior
- Needs both preconditions and a trigger -> Complex

## Characteristics of a Good Requirement

Every requirement you write or review must satisfy all of these:

| Characteristic | Description |
|---|---|
| **Unambiguous** | Has only one interpretation |
| **Traceable** | Has a unique identifier |
| **Consistent** | Does not conflict with other requirements |
| **Verifiable** | It is possible to check that the system meets the requirement |
| **Complete** | Not lacking relevant information |

## Troubleshooting

- **No sentence type fits:** Use a higher abstraction level until it makes sense, or ask the user for more information from the relevant stakeholder.
- **Cannot identify the actor:** Common with nonfunctional requirements. Express as "the system shall be ...".
- **No system response:** Common with nonfunctional requirements. Try "shall be immune" or similar workaround.
- **Need "shall not":** EARS deliberately avoids "shall not". Try "shall be immune" or similar. Use "shall not" only as a last resort.
- **Too many atomic requirements:** Deep technical requirements are not well suited to EARS. Consider using a list as accompaniment or another format if EARS seems inappropriate.

## Quick Checklist

- Actor named ("The <system>...")
- Correct pattern for the trigger/condition
- Conditions are specific (state names, thresholds)
- Response is observable/verifiable
- Avoids vague terms ("fast", "user-friendly", "appropriate")

## Quick Templates (Copy/Paste)

- Ubiquitous: The <system name> shall <system response>.
- Event-driven: When <optional preconditions> <trigger>, the <system> shall <system response>.
- State-driven: While <in a state>, the <system> shall <system response>.
- Optional feature: Where <feature is included>, the <system> shall <system response>.
- Unwanted behavior: If <optional preconditions> <trigger>, then the <system> shall <system response>.
- Complex: Where <feature is included>, while <state>, when <trigger>, the <system> shall <system response>.

## Reference Material

The full EARS quick reference document is available at `reference/ears-reference.adoc` within this skill's directory. Consult it for additional detail on patterns, checklists, troubleshooting, and good practices.

For the original publication, see https://ieeexplore.ieee.org/document/5328509

## When to use me

Use this skill when:
- Writing new requirements for a system or feature
- Translating informal or poorly structured requirements into EARS format
- Reviewing existing requirements for EARS compliance and quality
- Identifying missing requirements from a set of states, events, or features

Always ask clarifying questions if the system boundary, actors, or intended behavior are unclear.
