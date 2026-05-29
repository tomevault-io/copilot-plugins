## go-processmanager

> - `go test ./...` — run all tests (ginkgo suite via gomega assertions)

# go-processmanager

## Commands

- `go test ./...` — run all tests (ginkgo suite via gomega assertions)
- `go build ./...` — build the package
- `go vet ./...` — static analysis

Go 1.24+ required (`toolchain go1.24.11` in `go.mod`).

## Structure

Single package, no submodules. All source is flat in the root:

| File | Purpose |
|---|---|
| `config.go` | `Config` struct + `DefaultConfig()` + `Option` type |
| `options.go` | Functional options (`WithName`, `WithArgs`, etc.) |
| `process.go` | Core `Process` type: `Run()`, `Stop()`, `IsAlive()`, `Done()`, `ExitCode()` |
| `writer.go` | `NewLog(path)` opens an append-mode log file |
| `process_unix.go` | Unix build: process groups, signal sending, zombie reaping (`//go:build unix`) |
| `process_windows.go` | Windows build: no-op reaping, direct Kill via `os.FindProcess` (`//go:build windows`) |
| `subreaper_linux.go` | Linux-specific `SetSubreaper()` via `unix.Prctl(PR_SET_CHILD_SUBREAPER)` (`//go:build linux`) |
| `subreaper_other.go` | No-op `SetSubreaper()` on non-Linux (`//go:build !linux`) |
| `process_suite_test.go` | Ginkgo test runner entrypoint |
| `process_test.go` | All ginkgo specs |

## Architecture notes

- The package is `process`. It uses OS-specific build tags (`unix`, `windows`, `linux`, `!linux`) for platform behavior — don't assume Unix semantics apply everywhere.
- Process state (PID, stdout, stderr, exitcode) lives in a `StateDir` directory. Use `WithTemporaryStateDir()` for ephemeral instances.
- `KillSignal` defaults to SIGKILL (9). Set it via `WithKillSignal()`.
- `GracefulTimeout` defaults to 15s: SIGTERM → wait → SIGKILL if still alive.
- `KillProcessGroup` defaults to `true`. On Unix, negative PID is passed to `syscall.Kill()` to target the whole group.
- `monitor()` runs as a goroutine after `Run()`, closes the `done` channel on exit, and spawns `reapChildren()`.

## Testing quirks

- Tests use ginkgo/gomega with dot-imports (`. "github.com/onsi/ginkgo"`). Inside specs you call `Expect(...)` directly — no `g.Expect`.
- Tests use `Eventually()` with 2m timeouts in several places. They are not fast.
- `process_test.go` is package `process_test` (black-box testing of exported API).
- Each test creates a temp dir and cleans it up; don't rely on state directory persistence between tests.

---
> Source: [stnmrshx/go-processmanager](https://github.com/stnmrshx/go-processmanager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
