---
name: proven-needs
description: Spec-driven development workflow for creating production-grade software. Provides the overall pipeline, artifact lifecycle, and conventions shared by all needs-* skills. Every workflow skill (needs-user-story, needs-spec, needs-design, needs-adr, needs-architecture, needs-tasks, needs-implementation) loads this skill first. Also use when asked about the development workflow, artifact locations, how the skills connect, or the overall process.
---

## Pipeline

```
Feature Description -> User Stories -> Specifications -> Design -> Tasks -> Implementation
                       (needs-user-story)  (needs-spec)     |      (needs-tasks) (needs-implementation)
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
6. **Tasks** (`needs-tasks`) -- create a phased implementation task list from the design document. Tasks are organized into sequential phases with parallelism markers and full traceability back to specs and stories. Output: `docs/tasks/`.
7. **Implementation** (`needs-implementation`) -- implement the code phase by phase from the task list, using the design document for architectural guidance. Verifies, commits, and asks the user before proceeding to each next phase. Output: working code + updated `docs/tasks/` and `docs/design/` statuses.

Supporting skill: `ears-requirements` provides the EARS methodology used by steps 1 and 2.

## Artifacts

| Artifact | Location | Produced by | Lifecycle |
|---|---|---|---|
| User Stories | `user-stories.adoc` | `needs-user-story` | Living, SemVer versioned |
| Specifications | `docs/specs/` | `needs-spec` | Living, synced with user stories |
| Design Document | `docs/design/` | `needs-design` | Ephemeral -- stale when stories/specs change, invalid after implementation |
| ADRs | `docs/adrs/NNNN-title.adoc` | `needs-adr` | Permanent, append-only |
| Architecture | `docs/architecture.adoc` | `needs-architecture` | Living, reflects current system state |
| Tasks | `docs/tasks/` | `needs-tasks` | Ephemeral -- stale when design changes, status tracks completion |
| Implemented Code | project source | `needs-implementation` | Living -- the actual codebase produced from the task list |

### Staleness and validity

The design document tracks upstream versions and its own status:

- `:source-stories-version:` and `:source-specs-version:` -- which versions of stories and specs the design was built from
- `:status:` -- `Current` (valid), `Stale` (upstream changed), `Implemented` (design cycle complete)

Specifications track their upstream version:

- `:source-stories-version:` -- which version of user stories the specs were derived from

The architecture document tracks its upstream version:

- `:source-design-version:` -- which version of the design the architecture was generated from (omitted in reverse-engineer mode)

The task list tracks its upstream versions:

- `:source-design-version:` -- which version of the design the tasks were derived from
- `:source-stories-version:` and `:source-specs-version:` -- for full traceability
- `:status:` -- `Current` (valid), `Stale` (design changed), `Implemented` (all tasks ticked)

**Note:** Staleness is detected when a downstream skill is re-invoked and compares version numbers. The `:status:` field is not automatically set to `Stale` when upstream artifacts change -- it remains at its last-written value until the relevant skill re-evaluates. To verify the full artifact chain, re-run each downstream skill in pipeline order; it will detect and report any version mismatches.

## Skill Dependencies

```
proven-needs            <-- loaded by all workflow skills
├── needs-user-story    <-- also loads ears-requirements
├── needs-spec          <-- also loads ears-requirements
├── needs-design        <-- also loads needs-adr (for creating ADRs during design)
├── needs-adr
├── needs-architecture
├── needs-tasks
└── needs-implementation
```

Each workflow skill loads `proven-needs` first to understand the overall pipeline and conventions. Individual skills load additional prerequisites as needed (noted in their own SKILL.md).

### Ordering

The recommended pipeline order runs every step for maximum traceability:

1. `needs-user-story` -- no upstream dependency (takes feature description as input)
2. `needs-spec` -- requires `user-stories.adoc`
3. `needs-design` -- requires `user-stories.adoc`; uses `docs/specs/` when available
4. `needs-adr` -- no strict ordering (standalone or invoked from `needs-design`)
5. `needs-architecture` -- requires `docs/design/` (greenfield) or codebase (existing)
6. `needs-tasks` -- uses `docs/design/` when available; falls back to user stories
7. `needs-implementation` -- uses `docs/tasks/` when available; falls back to design's Story Resolution

If a required input is missing, the skill warns the user about the traceability impact and asks whether to proceed in reduced mode or create the missing artifact first.

### Fast-Track Paths

Not every project needs every step. The pipeline supports fast-tracking by skipping intermediate steps. User stories are always required -- they are the root input for every path.

| Path | Skills Run | Skipped | Traceability Impact |
|---|---|---|---|
| **Minimal** | user-story -> design -> implementation | spec, adr, architecture, tasks | No spec IDs. No task phasing. Stories resolved directly to code. |
| **Without specs** | user-story -> design -> tasks -> implementation | spec, adr, architecture | No spec-level traceability. Design and tasks reference only story IDs. |
| **Without tasks** | user-story -> spec -> design -> implementation | adr, architecture, tasks | Full spec traceability. No phased execution -- implementation follows story resolution order. |
| **Standard** | user-story -> spec -> design -> tasks -> implementation | adr, architecture | Full traceability. Technology decisions undocumented. |
| **Full** | All skills | Nothing | Complete traceability and documentation. |

ADRs and Architecture are always optional -- no downstream skill has a hard dependency on them. They can be added at any point.

**Reduced mode convention:** When a skill runs without an optional upstream artifact, it sets the corresponding `:source-*-version:` attribute to `n/a` in the output artifact. This signals to downstream skills and humans that the full pipeline was not followed and certain traceability links are unavailable. Quality checklist items that depend on the skipped artifact become conditional.

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

## Bootstrap

When this skill is loaded, **immediately** check the project's `AGENTS.md` for the proven-needs workflow marker. This ensures every project that uses the workflow is permanently configured.

### Steps

1. Read `AGENTS.md` in the project root (it may not exist yet).
2. Search for the marker `<!-- proven-needs:start -->`.
3. **If the marker is found** -- do nothing, the project is already bootstrapped.
4. **If the marker is NOT found** -- append the following block to `AGENTS.md` (create the file if it doesn't exist):

```markdown
<!-- proven-needs:start -->
## Development Workflow
This project follows the proven-needs spec-driven development (SDD) workflow.
Before implementing any feature or making significant changes, load the
`proven-needs` skill to understand the required pipeline:
User Stories → Specifications → Design → ADRs → Architecture → Tasks → Implementation.
<!-- proven-needs:end -->
```

5. Inform the user that `AGENTS.md` was updated and should be committed.

### Rules

- This check runs **every time** the skill is loaded, but is idempotent.
- Append to the **end** of the file to avoid disrupting existing content.
- Do NOT modify content between the markers if the block already exists.
- Perform this check **before** proceeding with any other workflow task.
