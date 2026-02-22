# Agents

This is the **proven-needs skill definition repository**. It contains the skill definitions, examples, and documentation for the proven-needs spec-driven development (SDD) pipeline.

## Working on this repo

When editing skills in this repository:

- **Keep the README in sync.** The `README.md` must accurately reflect the workflow, skills, and artifact lifecycle defined in `skills/proven-needs/SKILL.md`. When the pipeline, skill descriptions, or artifact table in `proven-needs` changes, update the README to match.
- **Maintain example consistency.** All example files (`references/example.adoc`) use the same e-commerce scenario. Story names, spec IDs, and design references must be consistent across all examples. The canonical story names are defined in `skills/needs-user-story/references/example.adoc`.
- **Preserve version tracking conventions.** Every artifact template uses `:source-*-version:` attributes to track upstream dependencies. When adding or modifying templates, follow the existing naming pattern (e.g., `:source-stories-version:`, `:source-specs-version:`, `:source-design-version:`). Use `n/a` for skipped upstream artifacts in fast-track paths.
- **Cross-reference accuracy.** Skills reference each other by name (e.g., "create specifications first using the `needs-spec` skill"). When renaming a skill or changing its responsibilities, search all other skills for references.
- **Guard clause pattern.** Each skill has exactly one hard stop on its immediate upstream input and soft guards on transitive upstream artifacts. Hard stops use "Stop and inform." Soft guards use "Warn and ask whether to proceed." Do not add new hard stops on non-immediate upstream artifacts.
