# Security Baseline

Apply on every task. Keep it short and actually applied.

- **Input:** validate at trust boundaries (request shape in the HTTP layer; business rules in the service).
- **AuthN/AuthZ:** middleware establishes identity; services decide whether that identity may perform the operation. Check where applicable.
- **Queries:** parameterize. Never build SQL by string concatenation with input.
- **Errors:** wrap internally; return generic messages for unexpected errors. Driver and internal detail goes to the log, never the client.
- **Secrets:** never in source, responses, or logs. Do not log tokens, credentials, or sensitive payloads.
- **Limits:** bound request body size (`http.MaxBytesReader`), set server and client timeouts, and use context cancellation on external calls.
- **Files and paths:** never join untrusted input into a filesystem path without validation. Avoid path traversal.
- **Dependencies:** check with project tooling when it exists. `govulncheck` and `gosec` are not mandatory in v1; run them if the project already uses them.

## Focused checks

One sensitive field or operation does not make a security-critical workflow. Run a short targeted check instead of the deep review.

| Field or operation | Check |
|---|---|
| Money amount stored or returned (e.g. product `price`) | No `float64` for exact values. Use integer minor units or a decimal type. Non-negative, range, overflow. State currency semantics if relevant. |
| Email, name, free text | Trim, length bound, character policy at the boundary. |
| Internally generated file path | Confined to an allowed root. |
| One outbound call to a fixed, configured provider | Timeout, context, bounded response, no redirects to arbitrary hosts. |
| Newly added endpoint | Is it public or protected? If auth middleware is a stub or no-op, say so. Do not call the route protected. |

Rule: a sensitive data type alone does not trigger deep review. Escalate when the change controls access, ownership, identity, transfer, disclosure, or an irreversible effect involving it.

## Classify findings

- **Introduced by this change:** fix in the diff.
- **Inherited and exercised:** the feature depends on a flawed path. Warn. Block only if the feature makes it exploitable, or if the feature's security claim depends on it (e.g. a "protected" route behind a no-op middleware).
- **Pre-existing, out of scope:** record separately as a follow-up. Do not fix silently or expand the diff.

## Escalate

If the change touches a trigger in `security-deep-review.md`, load it and perform the deep review.
