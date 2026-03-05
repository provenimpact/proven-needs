---
name: needs-tests
description: Derive and generate tests from feature specifications. Use when the proven-needs orchestrator determines that a feature needs test coverage. Operates within a single feature package at docs/features/<slug>/. Translates black-box verification descriptions in spec.adoc into executable test cases using the project's test framework.
---

## Prerequisites

Invoked by the `proven-needs` orchestrator with feature context (slug, intent, current state).

## Observe

1. Read `docs/features/<slug>/spec.adoc`: all requirement IDs, EARS texts, types, verification descriptions. If missing, report to orchestrator -- tests cannot be derived without specs.
2. Read `docs/features/<slug>/design.adoc` for component structure, interfaces, data model (informs test setup and level). If missing, tests will be limited to black-box behavioral tests.
3. Read `docs/features/<slug>/user-stories.adoc` for narrative context.
4. Scan test directories for existing tests (match by slug or spec ID references). Note existing infrastructure (helpers, fixtures, factories).
5. Read `docs/constraints.adoc` for quality constraints (coverage thresholds, test requirements).
6. Detect test framework: Jest/Vitest/Mocha/Playwright/Cypress (JS/TS), built-in (Rust), testing/testify (Go), pytest/unittest (Python), RSpec/Minitest (Ruby). Note file locations, naming conventions, assertion style.

## Evaluate

| Condition | Action |
|---|---|
| No tests for this feature | Generate full test suite |
| Tests exist, spec updated | Generate for new reqs, update for modified |
| Tests exist, all spec IDs covered | Current, report to orchestrator |
| Tests exist, some spec IDs missing | Generate for uncovered requirements |

## Execute

### Derive test cases from specifications

**Every requirement in `spec.adoc` MUST have at least one test. No exceptions.** A requirement without a test blocks the feature from `Implemented`.

For each requirement:

#### 1. Analyze the requirement

| EARS Type | Test Pattern |
|---|---|
| Ubiquitous | Assert behavior holds unconditionally (on load, after nav, across states) |
| Event-driven | Trigger event → assert response (valid + invalid triggers) |
| State-driven | Enter state → assert behavior (test entry/exit boundaries) |
| Unwanted behavior | Trigger error → assert recovery/handling |
| Optional feature | Enable → assert present; disable → assert absent |

#### 2. Determine test level

| Level | When |
|---|---|
| Unit | Single component with clear I/O (from design) |
| Integration | Multiple components or persistence |
| End-to-end | Full user journey or UI interaction |

Use design's Story Resolution to identify components. Without design, default to integration/e2e.

#### 3. Write the test case

Each test must: reference spec ID in description, follow project conventions, include happy path from verification field, include at least one edge case derived from EARS type, use existing infrastructure.

### Organize test files

Follow project conventions. Default:
```
tests/features/<feature-slug>/
├── <component>.test.<ext>       # Unit
├── <feature-slug>.test.<ext>    # Integration
└── <feature-slug>.e2e.<ext>     # E2E
```

### Write test files

Header pattern:
```
// Feature: <slug>  |  Spec version: <version>  |  Source: spec.adoc
// Coverage: PROD-001, PROD-002, ...
```

Test block pattern:
```
describe("<PREFIX>-<NNN>: <summary>", () => {
  it("should <happy path from verification>", () => { ... });
  it("should <edge case from EARS type>", () => { ... });
});
```

### Verify

1. **Compile/parse check:** all test files must be syntactically valid. No import, type, or syntax errors.
2. **Do NOT expect runtime passes.** No implementation exists yet -- failing tests are expected.
3. **100% spec coverage:** every spec ID MUST have a test. Missing = go back and generate it.

## Quality Checklist

- Every spec ID has at least one test case -- no exceptions
- Test descriptions reference spec IDs
- Tests follow project conventions
- Edge cases and error scenarios covered (especially unwanted-behavior EARS types)
- No duplicate helpers or fixtures
- All test files compile without errors
- Coverage constraints from `docs/constraints.adoc` satisfied
- Spec version recorded in test file headers

## Reference

See `references/example.adoc` for a complete example.
