---
name: needs-security
description: Assess and remediate the project security posture. Use when the proven-needs orchestrator determines that security improvements are needed. Operates at the project level. Observes dependency vulnerabilities, code patterns, and configuration issues. Proposes and applies remediations satisfying security constraints.
---

## Prerequisites

Invoked by the `proven-needs` orchestrator with desired state and current state context.

## Observe

### 1. Dependency vulnerabilities

Run the appropriate audit tool (`npm audit`, `cargo audit`, `pip-audit`, `bundle-audit`). Collect CVEs with severity levels.

### 2. Code pattern analysis

| Pattern | Risk | Detection |
|---|---|---|
| Hardcoded secrets | High | API keys, passwords, tokens in source |
| SQL injection | High | String concatenation in queries |
| Insecure deserialization | High | Unvalidated external data |
| Missing input validation | Medium | User inputs passed directly to logic |
| Insecure defaults | Medium | Default passwords, debug modes |
| Missing authentication | Medium | Endpoints without auth checks |
| Sensitive data in logs | Medium | PII or credentials logged |
| Insecure crypto | Medium | Weak algorithms (MD5/SHA1), small keys |

### 3. Configuration review

CORS settings, HTTPS enforcement, cookie security flags, rate limiting, auth configuration.

### 4. Read constraints

Read `docs/constraints.adoc` for security constraints.

## Evaluate

| Desired State | Actions |
|---|---|
| "No known vulnerabilities" | Patch deps (delegate to `needs-dependencies`), remediate code |
| "No hardcoded secrets" | Move to env vars or secret management |
| "All user input validated" | Add validation layers |
| "Security best practices" | Address all findings by severity |

Prioritize: Critical → High → Medium → Low.

## Execute

### 1. Dependency remediations

Delegate to `needs-dependencies` via the orchestrator. This skill focuses on code and configuration.

### 2. Code remediations

Apply fixes (move secrets, add validation, fix injection, etc.). Verify no breakage.

### 3. Configuration remediations

Apply secure configuration. Document changes.

### 4. Verify

Re-run security scans, run build and test suite, verify all security constraints satisfied.

## Reference

See `references/example.adoc` for an example.
