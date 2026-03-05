---
name: needs-dependencies
description: Manage and update the project dependency graph. Use when the proven-needs orchestrator determines that dependencies need updating, patching, or auditing. Operates at the project level. Observes outdated packages, vulnerabilities, unmaintained packages, and license compliance. Executes minimal updates respecting all constraints.
---

## Prerequisites

Invoked by the `proven-needs` orchestrator with desired state and current state context.

## Observe

### 1. Detect package manager

| File | Manager |
|---|---|
| `package.json` + `package-lock.json` | npm |
| `package.json` + `bun.lock` | bun |
| `package.json` + `yarn.lock` | yarn |
| `package.json` + `pnpm-lock.yaml` | pnpm |
| `Cargo.toml` | cargo |
| `go.mod` | go modules |
| `pyproject.toml` / `requirements.txt` | pip / poetry / uv |
| `Gemfile` | bundler |

### 2. Analyze dependency state

For each dependency: current version, latest version, version constraint, update type (patch/minor/major), vulnerability status (CVEs via `npm audit`/`cargo audit`/`pip-audit`/etc.), maintenance status, license.

### 3. Read constraints

Read `docs/constraints.adoc` for security constraints (CVE timelines) and licensing constraints.

## Evaluate

| Desired State | Actions |
|---|---|
| "No known vulnerabilities" | Update vulnerable deps to patched versions |
| "All dependencies up to date" | Update all outdated deps |
| "No unmaintained dependencies" | Identify replacements |
| "License compliance" | Replace or remove violations |
| Specific package update | Update the named dependency |

Risk levels follow the orchestrator's classification (patch=low, minor=medium, major=high). Patch updates are auto-approve eligible. Major updates and replacements require user approval.

## Execute

### 1. Apply updates

For each approved update: update version in manifest, run install/update, verify lockfile.

### 2. Verify

Run build, linter, test suite. Roll back problematic updates if verification fails.

### 3. Constraint re-check

Re-run vulnerability audit, re-check license compliance, confirm no new violations.

## Reference

See `references/example.adoc` for an example.
