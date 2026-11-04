---
name: go-backend
description: Write, change, debug, refactor, or review Go backend code (HTTP APIs, services, repositories, workers, migrations) following Clean Architecture, TDD, and a security baseline. Use when the task is a Go server-side service or an existing Go backend repository - API endpoint, handler, service, repository, middleware, new Go service, Go bug fix, Go code review. Do NOT use for standalone Go CLIs, scripts, or utilities, non-Go code, or snippets with no backend service. Adapts to the existing repository's conventions; applies a minimal Clean Architecture layout only for greenfield projects.
---

# Go Backend

Classify the task, load only the references it needs, run the matching workflow, report honestly.

**Out of scope:** standalone Go CLIs, scripts, or utilities, and any non-Go code. Do not load this skill or apply its approval gates, architecture, or references to them; just answer directly.

**Core principle:** the smallest structure that preserves correct dependency direction. No speculative layers, packages, or dependencies. An existing repository's conventions always win over the defaults here.

## 1. Inspect first

Read `references/repository-detection.md` and inspect the repository before proposing anything. Then classify:

- **Repository:** greenfield or existing.
- **Task:** feature, bug fix, simple change, refactor, review.
- **Risk:** does it touch a deep-review trigger (see `references/security-deep-review.md`)?

If the classification is ambiguous, ask. Do not guess.

## 2. Load references

| Reference | Load when |
|---|---|
| `go-standards.md` | every task |
| `repository-detection.md` | every task |
| `security-baseline.md` | every task |
| `verification.md` | any code-changing task |
| `tdd-workflow.md` | new behavior, bug fix, refactor |
| `clean-architecture.md` | greenfield, new boundaries, refactor, architecture questions |
| `api-conventions.md` | HTTP/API work |
| `security-deep-review.md` | only when a risk trigger appears |

Load on demand. Do not read everything up front.

## 3. Workflow by task type

### Greenfield
1. Clarify requirements; identify the capabilities needed.
2. Propose the minimal structure from `clean-architecture.md`. Create no optional folders without a requirement.
3. Ask before choosing a router/framework or database tooling.
4. Ask before adding a linter.
5. Write tests first, implement the minimum, verify.

When proposing before implementation, state the test-first sequence explicitly: (1) red - the failing test you write first and why it fails, (2) green - the minimum code to pass, (3) refactor - what you clean up with tests green. Do not write or edit files until the user approves the open decisions.

### New feature
1. Inspect existing boundaries; identify the affected capability.
2. If behavior or boundaries are unclear, propose a short design and wait for approval.
3. Write a failing test, implement the minimum, apply the security baseline, verify.

### Bug fix
1. Inspect implementation and tests; reproduce or establish the failure.
2. Find the root cause; re-read the relevant references.
3. **Propose the fix and wait for approval before editing.**
4. Write the regression test, implement the fix, verify.
5. No speculative refactoring.

### Simple change
Inspect local context, make the smallest safe change, add or update a focused test if behavior changes, run `gofmt` and applicable verification. No planning ceremony, no scaffolding, no deep review.

### Refactor
Confirm existing test coverage (add characterization tests if thin), preserve behavior, change one boundary at a time, rerun tests after each step, check dependency direction.

### Review
Read-only. Report findings by severity. No edits unless asked.

## Calibrate output to the task

Calibrate depth to the requested change's complexity, risk, and scope. A small feature, change, or bug fix needs focused inspection and the smallest safe workflow. A cross-capability, stateful, security-sensitive, or externally integrated feature needs deeper design and review. Do not apply architecture planning or deep review merely because the change touches an API endpoint.

| Task | Output |
|---|---|
| Simple change | Local inspection, smallest diff, verification. No architecture discussion. |
| Normal feature | Detected conventions, proposed files, TDD sequence, focused security checks, open decisions. |
| High-risk feature | Above, plus the deep security review. |
| Architecture request | Trade-offs, alternatives, dependency analysis. |

### Open decisions

Sort every open decision into one of three groups. Ask only about the first.

- **Blocking:** the answer changes a public contract or architecture and cannot be inferred. Examples: requested path conflicts with the repository's versioning, money representation, public vs authenticated endpoint, mixed response shapes, embedded vs separate storage. Keep to five or fewer.
- **Defaultable:** the repository already answers it. Examples: success status, migration naming, JSON key style, route registration, mock and test style. State the chosen default and continue.
- **Deferred:** out of scope. Examples: SKU, soft delete, event publishing, auth redesign. List briefly; do not ask.

Before asking anything, check the request against the repository for every contract conflict, and list all of them, not just the first. Always check: route prefix or version vs the repository's (`/v2/...` vs `/api/v1`), money type and unit in the request vs storage (float `12.99` vs int cents), and whether the required authorization mechanism (admin, role, ownership) exists. A missing auth mechanism does not replace the other conflicts: report every one under Blocking, each with a recommended option. Ask them all in ONE final message as a numbered list; never ask one and wait for the rest. Finding one conflict is not a reason to stop checking.

Report in this shape:

```text
Blocking decisions: ...
Defaults from repository: ...
Deferred as out of scope: ...
```

### Evidence and blocked inspection

Label repository facts as verified, inferred, or unknown (see `repository-detection.md`). If inspection is blocked, list the evidence needed and stop. Do not draft file paths or an implementation plan from guesses.

### Tests

Reuse the repository's existing test seam. Ask before adding a mocking library, test framework, or container tooling. If the repository has no tests, add one focused test at the narrowest stable boundary; do not build a testing architecture inside an unrelated feature.

## 4. TDD

New behavior follows red, green, refactor. If the `superpowers:test-driven-development` skill is available, use it and apply the Go specifics in `tdd-workflow.md` on top. If not, use the built-in loop in `tdd-workflow.md`.

## 5. Security

Apply `security-baseline.md` on every task, including its focused checks for a single sensitive field. Add the deep review only when a trigger in `security-deep-review.md` appears; the skill performs it directly, no separate agent.

Report findings as introduced by the change, inherited and exercised by the change, or pre-existing and out of scope. Do not silently fix the last group.

## 6. Verification

Follow `verification.md`. A failing project verification means the task is not done. Do not report success over failures, including pre-existing ones: diagnose and document them.

## 7. Report

State: changed files, tests added or updated, verification commands and results, security checks performed, known failures or risks, deferred decisions.

## Deferred decisions: stop and ask

- Use cases that span capabilities and need cross-capability consistency (transactions, events, compensation).
- DDD concepts (aggregates, value objects, bounded contexts). Clean Architecture is the v1 baseline. If a domain looks complex, say so and ask before introducing DDD.
- Optional packages (`internal/storage`, `internal/logging`, `cmd/worker`, `api/`) without an explicit requirement.
- Router, database tooling, and linter choices in greenfield projects.
- A response schema that differs from the default or from the existing project.

## Evolution note

Starting without DDD and refactoring toward it later is allowed. The refactor must preserve behavior and keep tests passing before, after, and between steps. Public API behavior and response shapes stay unchanged.
