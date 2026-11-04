# TDD Workflow

## Delegation

If `superpowers:test-driven-development` is available, invoke it and apply the Go specifics below on top. Otherwise use the built-in loop.

## Built-in loop

1. **Red:** write a failing test for one behavior. Run it. Confirm it fails for the expected reason.
2. **Green:** write the minimum code to pass. Run it. Confirm it passes.
3. **Refactor:** clean up with tests green. Run the tests again.
4. Repeat per behavior. Run `gofmt` before the test run.

For a bug fix, the first test reproduces the bug. It must fail before the fix and pass after.

## Go specifics

| Layer | Approach |
|---|---|
| Service logic | Table-driven unit tests, `t.Run` per case |
| Boundaries | Hand-written fakes of consumer-owned interfaces. No mock generation by default |
| HTTP handlers | `net/http/httptest` |
| Repositories | Real database integration tests when persistence exists |
| Generated code (e.g. sqlc) | Do not test it directly |

- Test behavior through public APIs, not internals. Behavior-level tests survive refactors, including a later DDD move.
- Add interfaces only where substitution is useful.
- Integration tests may skip only through an explicit project mechanism (build tag, `-short`). Never hide failures silently.
- Concurrency-sensitive code: run with `-race`.
- Existing repo test tooling (testify, gomock, etc.): follow it.

## Table-driven shape

```go
func TestService_Create(t *testing.T) {
	tests := []struct {
		name    string
		input   CreateInput
		wantErr error
	}{
		{"valid", CreateInput{Email: "a@b.co"}, nil},
		{"duplicate", CreateInput{Email: "dup@b.co"}, ErrConflict},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			svc := NewService(newFakeRepo())
			_, err := svc.Create(context.Background(), tt.input)
			if !errors.Is(err, tt.wantErr) {
				t.Fatalf("got %v, want %v", err, tt.wantErr)
			}
		})
	}
}
```

## Refactor safety

Before refactoring, confirm coverage. If thin, add characterization tests that pin current behavior first.
