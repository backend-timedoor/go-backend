# Clean Architecture

Clean Architecture here means dependency direction, not folder count. DDD is out of scope for v1.

## Dependency direction

```text
http handler -> capability service -> consumer-owned interface (repository, store)
                                          ^
                          db / storage implementation
cmd/api/main.go wires everything.
```

- Services never import `net/http`, database drivers, or read environment variables.
- Interfaces are defined by the consumer (the service package), not the implementer.
- Infrastructure implements those interfaces and maps driver errors to capability sentinels.
- Handlers call services, never the database directly.
- A capability never reads another capability's tables. Cross-capability reads go through an exported interface. Use IDs, not shared entities.

## Default greenfield shape

Create only what the project needs.

```text
cmd/
  api/main.go
internal/
  config/
  <capability>/
    errors.go
    service.go
    repository.go        # only when a persistence boundary or test seam earns it
  http/
    middleware/
    routes.go
    v1/
      <capability>/
        handler.go
        request.go
        response.go
```

Added on demand only:

| Path | Add when |
|---|---|
| `internal/db/` | persistence exists (often generated access code) |
| `migrations/` | schema is managed through migrations |
| `cmd/worker/main.go` | a background worker is explicitly required |
| `api/` | OpenAPI, Swagger, or protobuf contracts are maintained |
| `internal/logging/` | custom logging is explicitly required |
| `internal/storage/` | file or object storage is explicitly required |

Simple project: `cmd/api/main.go`, `internal/config/`, `internal/http/`. Grow from there.

## Rules

- Capability packages, named after what they do. Modules (`domain/`, `app/`, `adapter/` per capability) are optional and used only when a capability has real complexity, independent ownership, or likely extraction.
- No empty layers. No file exists to match a diagram.
- `internal/db/` holds database access code only. Business behavior stays in the capability.
- One database. A capability uses its own tables only.
- Migrations live in `migrations/`, outside `internal/`, are sequential, and are immutable once deployed. Add a new migration instead of editing an applied one. Run them as an explicit release step, not silently at startup.
- Middleware lives in `internal/http/middleware/` and handles transport concerns: request ID, recovery, access logging, auth token parsing, CORS, rate limits, size and time limits. Business authorization lives in the service.
- Version at the HTTP boundary (`/api/v1`). Do not create `v2` preemptively.

## Complexity signals

If a domain shows many invariants, overloaded services, or shared domain language across workflows, say so and ask before introducing DDD concepts. Do not apply DDD because there are multiple entities.

Cross-capability consistency (e.g. place order and charge payment) is a deferred decision. Stop and ask.

## Evolution

A later move toward DDD or modules is allowed. It must preserve behavior, keep tests passing before and after each step, and leave public API behavior and response shapes unchanged.
