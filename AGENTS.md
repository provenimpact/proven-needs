# Agents

This is the **proven-capability** skill definition repository. It contains the skill definitions, examples, and documentation for the intent-driven state transition workflow.

## Working on this repo

When editing skills in this repository:

- **Keep the README in sync.** The `README.md` must accurately reflect the workflow, skills, and artifact lifecycle defined in `skills/proven-intent/SKILL.md`. When the state transition model, skill descriptions, or artifact table changes, update the README to match.
- **Maintain example consistency.** All example files (`references/example.adoc`) use the same e-commerce scenario. Story names, spec IDs, and design references must be consistent across all examples. The canonical story names are defined in `skills/needs-stories/references/example.adoc`.
- **Preserve version tracking conventions.** Every artifact template uses `:source-*-version:` attributes to track upstream dependencies. When adding or modifying templates, follow the existing naming pattern (e.g., `:source-stories-version:`, `:source-spec-version:`, `:source-design-version:`). Use `n/a` for skipped upstream artifacts.
- **Cross-reference accuracy.** Skills reference each other by name (e.g., "derive specifications using the `needs-spec` capability"). When renaming a skill or changing its responsibilities, search all other skills for references.
- **Observe/evaluate/execute pattern.** Every capability skill (needs-*) follows the three-phase pattern: Observe (assess current state in this domain), Evaluate (does the desired state require action? do constraints allow it?), Execute (make the minimum changes). Maintain this structure when adding or modifying capabilities.
- **Feature-scoped artifacts.** Feature capabilities (needs-stories, needs-spec, needs-design, needs-tasks, needs-implementation, needs-tests) operate within a single feature package at `docs/features/<slug>/`. They must not read or depend on other feature packages. Project-wide capabilities (needs-adr, needs-architecture, needs-dependencies, needs-security, needs-compliance) operate at the project level.
- **Constraint enforcement.** Every capability checks relevant constraints from `constraints.adoc` during its Evaluate phase. Constraint violations block transitions unless the user explicitly updates the constraint.
- **Orchestrator responsibility.** The `proven-intent` skill is the single entry point. It handles intent classification, feature decomposition, feasibility evaluation, transition planning, and capability invocation. Individual capabilities do not independently decide pipeline ordering -- the orchestrator derives it.
