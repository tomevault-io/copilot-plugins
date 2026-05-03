## sshthing

> - `cmd/sshthing/`: CLI entrypoint (`main.go`) for the `sshthing` binary.

# Repository Guidelines

## Project Structure & Module Organization

- `cmd/sshthing/`: CLI entrypoint (`main.go`) for the `sshthing` binary.
- `internal/app/`: Bubble Tea application model (state, update loop, navigation).
- `internal/ui/`: Lipgloss styling + view rendering (login, modals, main view).
- `internal/db/`: SQLCipher database initialization, migrations, CRUD.
- `internal/crypto/`: PBKDF2 + AES-GCM helpers for per-key encryption.
- `internal/ssh/`: `ssh` subprocess integration, temp key handling, key generation.
- `internal/sync/`: Git synchronization logic (export/import, encrypted JSON, conflict resolution).
- `internal/config/`: Configuration file handling (`config.json`) and defaults.
- Docs: `README.md`, `QUICKSTART.md`, `CHANGELOG.md`.

## Build, Test, and Development Commands

- Build (macOS/Linux): `go build -o sshthing ./cmd/sshthing`
- Build (Windows): `go build -o sshthing.exe ./cmd/sshthing`
  - Requires MSYS2/MinGW-w64 for CGO. See [BUILDING_WINDOWS.md](BUILDING_WINDOWS.md).
- Produces the local binary (not committed).
- Run (macOS/Linux): `./sshthing`
- Run (Windows): `.\sshthing.exe`
- Tests: `go test ./...`
  - If your environment blocks the default Go build cache, run:
    - `GOCACHE=$PWD/.gocache go test ./...`

## Coding Style & Naming Conventions

- Language: Go. Format with `gofmt` (required).
- Packages use lowercase names (`app`, `db`, `ssh`, `ui`, `crypto`).
- Keep functions small and avoid UI logic in `internal/db`/`internal/ssh`.
- Binary name is always `sshthing` (docs and scripts should match).

## Testing Guidelines

- Framework: Go’s standard `testing` package.
- Prefer table-driven tests for pure logic (e.g., filtering, validation).
- Test files use Go conventions: `*_test.go` next to the package under test.

## Commit & Pull Request Guidelines

- Commit messages are short, imperative, and scoped when useful (e.g., `Cleanup: …`, `Fix: …`).
- PRs should include:
  - A clear summary of user-visible changes.
  - Any relevant screenshots (TUI screenshots are helpful for UI changes).
  - Notes on migrations or data-impacting changes (DB schema, encryption, paths).
- Agent rule: do **not** commit or push to GitHub unless the repo owner explicitly asks you to.

## Security & Configuration Tips

- User data is stored at `~/.ssh-manager/hosts.db` (SQLCipher encrypted).
- SSH sessions may override Ghostty’s TERM; you can force a value with:
  - `SSHTHING_SSH_TERM=xterm-256color ./sshthing`

---
> Source: [Vansh-Raja/SSHThing](https://github.com/Vansh-Raja/SSHThing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
