---
name: ears-requirements
description: Write and review requirements using the EARS (Easy Approach to Requirements Syntax) methodology. Guides structured requirement authoring with correct sentence types, identifies missing requirements, and ensures quality characteristics.
---

## What I do

Guide the writing, review, and translation of requirements using the EARS (Easy Approach to Requirements Syntax) methodology.

## EARS Sentence Types

Every requirement must use one of the following sentence types. Choose the type based on the nature of the requirement.

### Ubiquitous

Requirements that apply at all times without a specific trigger or state.

**Template:** The `<system name>` shall `<system response>`.

**Example:** The kitchen system shall have an input hatch.

### Event-driven

Requirements triggered by a specific event, optionally with preconditions.

**Template:** When `<optional preconditions>` `<trigger>`, the `<system>` shall `<system response>`.

**Example:** When the chef inserts a potato to the input hatch, the kitchen system shall peel the potato.

### State-driven

Requirements that apply while the system is in a particular state.

**Template:** While `<in a state>`, the `<system>` shall `<system response>`.

**Example:** While the kitchen system is in maintenance mode, the kitchen system shall reject all input.

### Unwanted Behavior

Requirements that handle error conditions or undesirable inputs.

**Template:** If `<optional preconditions>` `<trigger>`, then the `<system>` shall `<system response>`.

**Example:** If a spoon is inserted to the input hatch, then the kitchen system shall eject the spoon.

### Optional

Requirements that only apply when a specific feature is present.

**Template:** Where `<feature>`, the `<system>` shall `<system response>`.

**Example:** Where the kitchen system has a food freshness sensor, the kitchen system shall detect rotten foodstuffs.

### Combined Sentences

Sentence types can be combined for complex requirements. Use the keywords in this order when combining: **Where** ... **While** ... **When** ... / **If** ... **then** ...

**Example:** Where the car has an ABS system, while the car is moving, when the driver applies brake, the ABS system shall detect blocked wheels.

## How to Apply EARS

Follow these steps when writing or translating requirements:

1. **Identify** whether you are working with a requirement, or something else (e.g. a note, example, or design decision). Only requirements get EARS treatment.
2. **Identify compound requirements** -- determine whether the requirement needs to be split into multiple atomic requirements.
3. **Identify the acting system, person, or process.**
4. **Analyse the needed sentence type(s)** -- select from Ubiquitous, Event-driven, State-driven, Unwanted Behavior, or Optional.
5. **Identify possible missing requirements** -- e.g. 2 states and 2 events usually produce 4 requirements. Check all combinations.
6. **Analyse the translated requirements** for ambiguity, conflict, and repetition.
7. **Review requirements** if possible. Iterate as required.

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

## Reference Material

The full EARS quick reference document is available at `reference/ears-reference.adoc` within this skill's directory. Consult it for additional detail on templates, combined sentences, troubleshooting, and good practices.

## When to use me

Use this skill when:
- Writing new requirements for a system or feature
- Translating informal or poorly structured requirements into EARS format
- Reviewing existing requirements for EARS compliance and quality
- Identifying missing requirements from a set of states, events, or features

Always ask clarifying questions if the system boundary, actors, or intended behavior are unclear.
