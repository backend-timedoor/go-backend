# Repository Detection

Inspect before proposing changes. Preserve what works.

## Read

| Source | Learn |
|---|---|
| `go.mod`, `go.work` | Go version, module path, dependencies, workspace |
| `Makefile`, CI config, `Dockerfile` | Verification contract, build and deploy constraints |
| Directory tree (`cmd/`, `internal/`, `pkg/`) | Package layout and conventions |
| Imports and `go.mod` | Router or framework, database tooling (`database/sql`, `pgx`, `sqlc`, GORM, ent), validation library |
| `.golangci.yml` / `.golangci.yaml` | Whether a linter is configured |
| `*_test.go` and test helpers | Test style: table-driven, testify, fakes, mocks, integration tags |
| Existing handlers and `response` code | The project's response envelope |
| `migrations/`, `sqlc.yaml` | Schema management and generated code |

## Decide

- **Greenfield:** no `go.mod`, or an empty skeleton. Apply the defaults in `clean-architecture.md`.
- **Existing:** follow its structure, naming, router, database tooling, test style, and response shape. Do not restructure without need.
- **Established router or database tooling:** use it without asking.
- **Multiple or unclear routers, tooling, or response shapes:** inspect usage; ask which is authoritative.

## Evidence confidence

Label each repository fact you rely on:

- **Verified:** read directly from source.
- **Inferred:** deduced from neighboring code, naming, or a summary.
- **Unknown:** not inspected, or inspection was blocked.

Rules:

- Do not turn an inferred convention into a proposed file path without labeling it.
- Do not claim the repository's conventions were followed when the relevant files were unavailable.
- If inspection is blocked (no clone, no network, no access), list the evidence needed and stop. Do not draft a pseudo-implementation.
- Summaries from tools or fetches are inferred, not verified. Re-check against source before editing.

## Conflicts between requests and the repository

Precedence: explicit user contract, existing public API contract, repository convention, skill default (see `api-conventions.md`).

When the first two conflict, for example a requested `/api/v1/...` in a repository that mounts routes at `/api/...`, do not silently rewrite the request and do not version the whole API. Ask one precise question: apply the requested path to this endpoint only, follow the repository convention, or was the path illustrative.

Do not re-ask what the repository already answers. State the default and continue.

## Test seam

Find the existing seam first: interfaces, fakes, testify mocks, integration tags, or none. Reuse it. If there are no tests, propose the smallest focused test at a stable boundary and ask before adding a new test dependency.

## Do not

- Re-scaffold a project that already has structure.
- Mark a project greenfield because it is small.
- Rely on a stale marker file; inspect the repository each time.
- Introduce a new dependency to match the skill's preference.
