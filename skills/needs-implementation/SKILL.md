---
name: needs-implementation
description: Implement code from the phased task list produced by needs-tasks. Use when asked to implement the tasks, start coding, build the feature, execute the implementation plan, or begin development. Reads docs/tasks/tasks.adoc and docs/design/ to produce working code, one phase at a time. After each phase, verifies the build, commits, and asks the user before proceeding to the next phase. This is the final step in the proven-needs pipeline.
---

## Prerequisites

Load the `proven-needs` skill first. It provides the overall workflow context, artifact locations, and conventions.

## Workflow

Implementing from the task list involves these steps:

1. Read inputs
2. Check for existing progress
3. Build phase todo list
4. Implement current phase
5. Verify phase
6. Commit phase
7. Ask to continue

Steps 3--7 repeat for each phase.

### 1. Read Inputs

Read these sources in order:

1. **`docs/tasks/tasks.adoc`** -- primary driver. Extract `:version:`, `:status:`, all phases, all tasks with their metadata (Components, Stories, Specs, Description, parallel/sequential markers). **If missing:** Stop and inform the user that tasks must be created first using the `needs-tasks` skill.

2. **If `:status:` is `Stale`:** Warn the user that the task list is stale (design has changed). Ask whether to proceed anyway or update the task list first using `needs-tasks`.

3. **If `:status:` is `Implemented`:** Inform the user all tasks are already complete. Ask whether to force re-implement or stop.

4. **`docs/design/design.adoc`** -- implementation reference. Extract system design sections, story resolution mappings, and architectural decisions. Also read `data-model.adoc` and files in `contracts/` if they exist. **If missing:** Warn the user that implementation will lack architectural guidance. Ask whether to proceed using only the task list or create the design first using the `needs-design` skill.

5. **`user-stories.adoc`** and **`docs/specs/`** -- for context on acceptance criteria and testable requirements. **If missing:** Note that story/spec context is unavailable. Proceed using the task list and design document.

6. **Existing codebase** -- analyze current code structure, existing patterns, frameworks, and conventions to ensure new code is consistent.

### 2. Check for Existing Progress

Examine `docs/tasks/tasks.adoc` for tasks already ticked `[x]`.

**If no tasks are ticked:** Start from Phase 1.

**If some tasks are ticked:** Identify which phases are fully complete (all tasks ticked) and which phase is the first with unticked tasks.

| Condition | Action |
|---|---|
| All tasks in phases 1--N are ticked, phase N+1 has unticked tasks | Inform user of progress. Ask whether to resume from phase N+1 or restart from the beginning (warn about overwriting existing code). |
| A phase has a mix of ticked and unticked tasks | Inform user of partial progress within the phase. Ask whether to continue the partial phase (implementing only unticked tasks) or redo the entire phase. |

### 3. Build Phase Todo List

Parse the tasks belonging to the **current phase only** from `docs/tasks/tasks.adoc`. Create a tracking list for the current phase's tasks. If a todo-list tool is available (e.g., TodoWrite), use it for progress tracking. Otherwise, track progress by ticking tasks directly in `docs/tasks/tasks.adoc`.

- Each task title from the phase becomes a todo item, prefixed with its task ID (e.g., "TASK-001: Initialize Next.js project with TypeScript configuration")
- All items start as `pending`
- Do **not** include tasks from other phases

When moving to a subsequent phase, replace the tracking list with a fresh one containing only the next phase's tasks.

### 4. Implement Current Phase

Work through the tasks in the phase, following the task list to the letter. Do not skip, reorder, or add tasks.

**For each task:**

1. Mark the task as `in_progress` in the todo list
2. Read the design document sections referenced in the task's `Components::` field to understand the architectural intent
3. Implement the code -- write files, create modules, set up configuration as described in the task's `Description::` field and informed by the design document
4. Mark the task as `completed` in the todo list
5. Tick the task `[x]` in `docs/tasks/tasks.adoc`

**Ordering within a phase:**

- **`[sequential]`** tasks: implement in order, one after another
- **`[parallel]`** tasks: may be implemented in any order (they have no intra-phase dependencies)

**Implementation principles:**

- Follow the design document for architecture, component structure, data model, and interface contracts
- Follow the task description for scope -- implement exactly what the task describes, nothing more
- Match existing codebase conventions (naming, file structure, patterns, formatting)
- Write production-quality code -- no placeholders, no TODOs, no stubs (unless the task explicitly calls for a placeholder to be wired in a later phase)

### 5. Verify Phase

After all tasks in the current phase are implemented:

1. Detect the project's verification commands by checking for `package.json` (scripts: build, test, lint, typecheck), `Makefile`, `Cargo.toml`, `pyproject.toml`, or similar configuration files
2. Run the build/compile step
3. Run the linter/type checker if available
4. Run the test suite if available
5. If any verification step fails, fix the issues before proceeding -- do not move on with a broken build

### 6. Commit Phase

After verification passes, commit the phase's work:

1. Stage all new and modified files relevant to this phase
2. Create a commit with a message following the pattern: `feat(<scope>): implement phase N -- <Phase Name>` followed by a body listing the task IDs completed
3. If the commit fails due to GPG signing (passphrase not cached), inform the user: "Commit failed -- GPG passphrase may not be cached. Run `gpg-connect-agent reloadagent /bye` or sign something manually to cache your passphrase, then tell me to retry." Wait for the user to confirm before retrying.

### 7. Ask to Continue

Use the question tool to ask the user whether to proceed.

**If more phases remain:**

Present a question with options:
- "Continue to Phase N: \<Phase Name\>" -- proceed to the next phase (loop back to step 3)
- "Stop here" -- halt implementation, progress is saved in `tasks.adoc`

**If all phases are complete:**

1. Set `:status: Implemented` in `docs/tasks/tasks.adoc`
2. Set `:status: Implemented` in `docs/design/design.adoc`
3. Update `:last-updated:` to today's date in both files
4. Commit these status updates
5. Inform the user that implementation is complete

## Reference

See `references/example.adoc` for a walkthrough showing how Phase 1 of the e-commerce task list becomes implemented code, continuing the example from the `needs-tasks` skill.
