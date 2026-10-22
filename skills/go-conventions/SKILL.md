---
name: go-conventions
description: Go build, test and lint commands plus idiomatic Go rules. Use when writing, reviewing or verifying Go code (go.mod, *.go, *_test.go).
---

# Go conventions

Repo `AGENTS.md`, `Makefile` and CI config win over these defaults.

## Commands

```sh
go build ./...
go test ./... -race -count=1
go vet ./...
gofmt -l .                 # should print nothing
goimports -l .             # if installed
golangci-lint run ./...    # if configured
```

Run a single test: `go test ./pkg/... -run TestName -v`.

## Rules

- Wrap errors with context: `fmt.Errorf("load config: %w", err)`. Compare with `errors.Is` / `errors.As`, never string matching.
- Pass `context.Context` as the first parameter; honor cancellation in loops and I/O.
- Do not panic in library code; return errors. `panic` only for programmer errors at init.
- Avoid goroutine leaks: every goroutine needs a stop path. Prefer `errgroup` over hand-rolled `WaitGroup` + channels.
- Accept interfaces, return concrete types. Keep interfaces defined at the consumer side, small (1–3 methods).
- Zero values should be usable. Prefer `time.Duration`/`time.Time` over raw ints/strings for time.
- Use `slog` for structured logging; no `fmt.Println` in production paths.
- Table-driven tests with `t.Run`; use `t.Parallel()` where safe; `t.Helper()` in helpers; `t.TempDir()` for files.
- Never ignore an error silently; if intentional, assign to `_` with a comment explaining why.
- `defer` inside loops: extract to a function to avoid holding resources.
- Struct tags and exported API changes are breaking changes — call them out.
