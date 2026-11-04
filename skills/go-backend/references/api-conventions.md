# API Conventions

## Precedence

Apply the first applicable source:

1. Explicit user- or developer-approved API contract.
2. Existing project's public API contract and established convention.
3. Greenfield OpenAPI, protobuf, API mock, or PRD.
4. The default below.

For a new project, inspect available OpenAPI/protobuf files, API mocks, and PRD/API examples before choosing endpoint paths, request/response shapes, status codes, or pagination. Treat an approved contract as authoritative; ask when sources conflict.

For an existing project, preserve its public API unless an explicit change is approved. If response shapes or contracts are mixed or unclear, ask which is authoritative. Do not guess.

A developer-proposed schema is an explicit contract only after confirmation. Ask once before it replaces an existing convention or the default.

When an explicit request conflicts with an existing public contract, do not silently rewrite either one: explain the conflict and ask whether to preserve the contract, add a versioned endpoint, or make a breaking change.

## Routing and versioning

- Version at the HTTP boundary: `/api/v1/...`. No preemptive `v2`.
- Handlers own version-specific DTOs. Share services across versions when behavior is compatible; split use cases only when business behavior differs.
- Prefer path versioning over header versioning unless an external contract requires it.
- Router or framework: existing repo uses its own; greenfield asks first.

## Default response envelope

JSON keys are `snake_case`.

Success:
```json
{ "data": {} }
```

Paginated (offset-based by default):
```json
{
  "data": [],
  "links": { "first": "", "last": "", "prev": null, "next": null },
  "meta": { "current_page": 1, "from": 1, "last_page": 1, "path": "", "per_page": 15, "to": 1, "total": 1 }
}
```

`links.prev` and `links.next` are `null` at the ends, not empty strings. Use cursor pagination only when the prompt, PRD, or API contract asks for it.

Error (4xx/5xx):
```json
{
  "error": {
    "code": "validation_failed",
    "title": "Validation Failed",
    "message": "The request is invalid.",
    "errors": [{ "field": "email", "message": "must be a valid email" }]
  }
}
```

- `code` is a stable machine-readable string, not the HTTP status.
- `errors` is `[]` when empty, never omitted.
- Envelope structs and helpers are shared in `internal/http/response.go`. Resource DTOs live in each capability's `response.go`.
- Services never see the envelope or status codes.

## Error mapping

| Sentinel | Status | `code` |
|---|---|---|
| `ErrNotFound` | 404 | `not_found` |
| `ErrConflict` | 409 | `conflict` |
| `ErrInvalidInput` | 422 | `validation_failed` |
| anything else | 500 | `internal_error` (generic message; detail to the log only) |

## Validation

- Existing repo: follow its approach.
- Greenfield default: `go-playground/validator` struct tags on request structs in `request.go`.
- Tags check shape and format at the transport boundary. Business rules ("email already taken", "order not cancellable") live in the service and return sentinels.
- Report `field` using the JSON name (`snake_case`), not the Go field name. Register a tag-name function.
- Map each tag (`required`, `email`, `min`, `max`) to a clean message. Never expose raw validator messages.
- Build one shared validator instance. Do not create one per request.
- Decode with `DisallowUnknownFields` and limit the body with `http.MaxBytesReader`.
- Validation failure returns 422 `validation_failed` with details in `errors[]`.
