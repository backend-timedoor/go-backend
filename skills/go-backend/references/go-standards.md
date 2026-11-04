# Go Standards

Existing repository conventions override everything here. These are defaults.

## Version
- Greenfield: target Go 1.23+.
- Existing repo: honor `go.mod`, `go.work`, CI, Docker, and deployment constraints.
- Never upgrade the Go version unless asked. Do not use APIs newer than the declared module version.

## Dependencies
- Standard library first.
- Add a dependency only with clear justification. Security or infrastructure dependencies need a stated rationale and verification.
- Do not upgrade dependencies unless asked or required.
- Existing lock and config files take precedence.
- Do not add a router, ORM, linter, logger wrapper, or storage SDK without need and approval.

## Naming and packages
- Packages: short, lowercase, one word, no underscores. Name after the capability, not `utils`, `common`, or `helpers`.
- Use the package name for context: `user.Service`, not `user.UserService`.
- Getters have no `Get` prefix: `Owner()`, not `GetOwner()`.
- Interfaces are small, usually one method. Define them in the consuming package, not the implementing one.
- Return concrete types from constructors; accept interfaces.

## Errors
- Return `error` as the last value. Never discard an error with `_` without a stated reason.
- Wrap with context: `fmt.Errorf("create user: %w", err)`.
- Match with `errors.Is` / `errors.As`. Never compare error strings.
- Sentinels live in `internal/<capability>/errors.go`: `ErrNotFound`, `ErrConflict`, `ErrInvalidInput`.
- Repositories map driver errors (no rows, unique violation) to capability sentinels. Driver errors never leave the persistence layer.
- Log an error once, at the boundary that handles it, not at every layer.
- Libraries do not `panic` for ordinary failures. `recover` only at deliberate process or goroutine boundaries.

## Context
- `context.Context` is the first parameter of anything that does I/O or blocks.
- Never store a context in a struct.
- Use context for cancellation and deadlines, not optional parameters.
- Derive timeout contexts where an operation needs its own bound; always `defer cancel()`.

## Concurrency
- Every goroutine has an owner, a termination path, and an error policy.
- No fire-and-forget goroutines in request handlers.
- Only the sending side that owns a channel closes it.
- Prefer channels for coordination; use a mutex where it is simpler for shared state.
- Concurrency-sensitive changes get `go test -race ./...`.

## Shutdown and server
- `cmd/api/main.go` owns signal handling (`SIGINT`, `SIGTERM`) and graceful shutdown.
- Shutdown stops accepting requests, calls `http.Server.Shutdown` with a timeout, closes workers and resources, and reports errors.
- Set read, write, and idle timeouts explicitly.

## Logging
- Use `log/slog` directly. Build the logger in `main.go` and pass it explicitly.
- No package-global mutable logger.
- Create `internal/logging/` only when the prompt, PRD, or design explicitly requires custom setup (redaction, multiple outputs, custom handlers).
- Request logging belongs in HTTP middleware. Domain code does not log transport details.

## Configuration
- Environment variables for deployment configuration.
- Typed config loaded once at startup in `internal/config`. Fail fast on missing required values.
- Safe defaults only for non-sensitive local settings.
- `.env.example` lists names only. Never commit `.env`.
- Config files only when requested or already established.
- Domain and service code never reads environment variables.

## Storage
- None by default. Add `internal/storage/` only when the prompt, PRD, or design names a need.
- Then ask which backend (local, S3, other). The consuming capability defines the small interface it needs; `storage/` implements it.

## Avoid
- DDD in v1; empty `domain/`, `application/`, `adapter/` layers.
- Scaffolding `worker`, `storage`, `logging`, `api`, or `migrations` without a requirement.
- Imposing the greenfield layout on an existing repo.
- Handlers reaching the database directly when a capability service exists.
- One capability touching another capability's tables.
- HTTP status codes or response envelopes inside services.
- Business state rules encoded only in validator tags.
- Logging secrets, tokens, credentials, or sensitive payloads.
- Cross-capability transaction or event patterns before a project requires them.
