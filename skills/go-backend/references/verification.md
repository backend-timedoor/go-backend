# Verification

A task is not done while required verification fails.

## Order

1. If the `Makefile` or documented CI defines a verification target, run it.
   - On failure, run the direct Go checks below diagnostically to find the root cause.
   - Direct-command success never overrides a failed project target. The task stays failed until the cause is fixed.
2. Otherwise run:
   ```text
   gofmt -w <changed .go files>
   go vet ./...
   golangci-lint run      # only if a linter config exists
   go test ./...
   ```
3. Concurrency-sensitive changes also run `go test -race ./...`.

## Linter

- Config exists: use it.
- New project without a linter: ask before adding or configuring one.

## Not mandatory in v1

`govulncheck` and `gosec`. Run them if the project already uses them.

## Failures

- Report the command and its output.
- Pre-existing failures: diagnose and document. Never silently bypass them.
- Do not claim success with failing or skipped checks; state what was skipped and why.
