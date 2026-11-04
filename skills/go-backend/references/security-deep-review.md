# Security Deep Review

Load only when a trigger appears. The skill performs this review directly; there is no separate reviewer agent in v1.

## Triggers

Authentication, sessions, or tokens. Payments or money operations. PII or multi-tenant data. File upload or path handling. Cryptography. Webhooks or outbound HTTP.

**Money operations** include charging, refunds, transfers, settlements, payouts, balance changes, financial accounting, or authorization of money movement. A product price stored or returned without money movement receives focused precision, range, overflow, and currency checks from `security-baseline.md`, not a full payment deep review.

Cheap code signals: `os.Open`, `filepath.Join` with input, `http.Client`, `crypto/*`, token parsing, tenant or user ID in queries.

## Checks by trigger

**Auth, sessions, tokens**
- Compare secrets and tokens in constant time. Do not roll your own crypto.
- Validate signature, expiry, issuer, and audience. Reject unsigned or `none`-algorithm tokens.
- Enforce authorization on every protected operation, not only at the route.
- Rate-limit login and recovery flows. Avoid account enumeration in error messages.

**Payments / money**
- Use integer minor units or a decimal type, never floats.
- Make charge operations idempotent. Verify amounts server-side.
- Do not trust client-provided prices or totals.

**PII / multi-tenancy**
- Every query is scoped by tenant. Verify the caller may access the resource ID (object-level authorization).
- Do not log PII. Minimize what responses return.

**Files / paths**
- Validate and clean paths; confine to an allowed root. Check size and content type; do not trust the client filename.
- Never serve user files from an executable location.

**Crypto**
- Use the standard library or vetted packages. Use `crypto/rand`, not `math/rand`, for secrets.
- Hash passwords with a purpose-built algorithm (bcrypt, argon2).

**Webhooks / outbound HTTP**
- Verify webhook signatures and replay windows.
- Guard against SSRF: restrict schemes, hosts, and private address ranges. Set timeouts. Limit redirects and response size.

## Output

Report findings by severity with the file, the issue, and a concrete fix. Do not claim safety beyond what was checked.
