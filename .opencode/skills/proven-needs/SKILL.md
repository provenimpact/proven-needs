---
name: proven-needs
description: Spec-driven development workflow for creating production-grade software. Provides the overall pipeline, artifact lifecycle, and conventions shared by all needs-* skills. Every workflow skill (needs-user-story, needs-spec, needs-design, needs-adr, needs-architecture) loads this skill first. Also use when asked about the development workflow, artifact locations, how the skills connect, or the overall process.
---

## Pipeline

```
Feature Description -> User Stories -> Specifications -> Design
                       (needs-user-story)  (needs-spec)     |
                                                        +----+----+
                                                        |         |
                                                      ADRs    Architecture
                                                   (needs-adr) (needs-architecture)
```

1. **User Stories** (`needs-user-story`) -- decompose a feature description into atomic user stories with EARS acceptance criteria. Output: `user-stories.adoc`.
2. **Specifications** (`needs-spec`) -- derive categorized, black-box testable EARS requirements from user stories. Output: `docs/specs/`.
3. **Design** (`needs-design`) -- design the implementation that solves the user stories, constrained by specs and ADRs. Identifies technology decisions and offers to create ADRs. Output: `docs/design/`.
4. **ADRs** (`needs-adr`) -- record significant technology decisions. Created during the design step or standalone. Output: `docs/adrs/`.
5. **Architecture** (`needs-architecture`) -- document the current system architecture. Generated from design (greenfield) or codebase (existing). Output: `docs/architecture.adoc`.

Supporting skill: `ears-requirements` provides the EARS methodology used by steps 1 and 2.

## Artifacts

| Artifact | Location | Produced by | Lifecycle |
|---|---|---|---|
| User Stories | `user-stories.adoc` | `needs-user-story` | Living, SemVer versioned |
| Specifications | `docs/specs/` | `needs-spec` | Living, synced with user stories |
| Design Document | `docs/design/` | `needs-design` | Ephemeral -- stale when stories/specs change, invalid after implementation |
| ADRs | `docs/adrs/NNNN-title.adoc` | `needs-adr` | Permanent, append-only |
| Architecture | `docs/architecture.adoc` | `needs-architecture` | Living, reflects current system state |

### Staleness and validity

The design document tracks upstream versions and its own status:

- `:source-stories-version:` and `:source-specs-version:` -- which versions of stories and specs the design was built from
- `:status:` -- `Current` (valid), `Stale` (upstream changed), `Implemented` (design cycle complete)

Specifications track their upstream version:

- `:source-version:` -- which version of user stories the specs were derived from

## Skill Dependencies

```
proven-needs            <-- loaded by all workflow skills
├── needs-user-story    <-- also loads ears-requirements
├── needs-spec          <-- also loads ears-requirements
├── needs-design        <-- also loads needs-adr (for creating ADRs during design)
├── needs-adr
└── needs-architecture
```

Each workflow skill loads `proven-needs` first to understand the overall pipeline and conventions. Individual skills load additional prerequisites as needed (noted in their own SKILL.md).

### Ordering

Skills must be run in pipeline order because each step depends on the output of the previous:

1. `needs-user-story` -- no upstream dependency (takes feature description as input)
2. `needs-spec` -- requires `user-stories.adoc`
3. `needs-design` -- requires `user-stories.adoc` and `docs/specs/`
4. `needs-adr` -- no strict ordering (standalone or invoked from `needs-design`)
5. `needs-architecture` -- requires `docs/design/` (greenfield) or codebase (existing)

If a required input is missing, the skill stops and directs the user to run the appropriate upstream skill first.

## Conventions

**Format:** All artifacts use AsciiDoc (`.adoc`).

**Versioning:** All versioned artifacts use SemVer (`MAJOR.MINOR.PATCH`):

| Change | Bump |
|---|---|
| Content removed or fundamentally rewritten | MAJOR |
| Content added or modified (non-breaking) | MINOR |
| Typos, formatting, metadata-only changes | PATCH |

**Dates:** Use `YYYY-MM-DD` format in all `:last-updated:` and `:date:` attributes.

**Requirement syntax:** All acceptance criteria and specifications use [EARS sentence types](ears-requirements). The `ears-requirements` skill provides the methodology reference.

**Black-box constraint:** Specifications (`docs/specs/`) describe only externally observable behavior. Internal architecture details belong in the design document (`docs/design/`) and architecture document (`docs/architecture.adoc`), never in specifications.
