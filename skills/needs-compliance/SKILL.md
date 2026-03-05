---
name: needs-compliance
description: Verify and enforce license and policy compliance across the project. Use when the proven-needs orchestrator determines that compliance needs checking or enforcement. Operates at the project level. Observes dependency licenses, identifies violations, proposes remediations.
---

## Prerequisites

Invoked by the `proven-needs` orchestrator with desired state and current state context.

## Observe

### 1. Dependency license analysis

For each dependency (direct and transitive): identify declared license, classify type.

| Category | Licenses | Typical constraint |
|---|---|---|
| Permissive | MIT, Apache-2.0, BSD-2/3-Clause, ISC | Generally allowed |
| Weak copyleft | LGPL-2.1/3.0, MPL-2.0 | May have linking restrictions |
| Strong copyleft | GPL-2.0/3.0, AGPL-3.0 | Often restricted in proprietary projects |
| Proprietary | Commercial | Requires license agreement |
| Unknown | No license declared | Risk -- may not be legally usable |

### 2. Read constraints

Read `docs/constraints.adoc` for licensing constraints.

### 3. Identify violations

Compare each dependency's license against constraints. Flag: direct violations, transitive violations, unknown licenses.

## Evaluate

| Desired State | Actions |
|---|---|
| "All dependencies license-compliant" | Replace or remove violating deps |
| "No unknown licenses" | Research and document, replace if non-compliant |
| "Full compliance audit" | Comprehensive report |

For each violation: is there a compliant drop-in replacement? Can the dep be removed? Is an exception warranted (requires constraint update)?

## Execute

### 1. Replace non-compliant dependencies

Remove non-compliant dep, add compliant replacement, update imports, run install.

### 2. Remove unnecessary non-compliant dependencies

Remove dep, replace with native/built-in alternatives or inline implementations.

### 3. Document exceptions

If user approves exception: record in `docs/constraints.adoc` under licensing with rationale.

### 4. Verify

Re-run license analysis, run build and tests, verify all licensing constraints satisfied.

## Reference

See `references/example.adoc` for an example.
