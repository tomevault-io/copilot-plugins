## diskclean-cli

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

**Build:**
```powershell
go build -o .\dist\diskclean.exe ./cmd/diskclean
```

Set version at build time:
```powershell
go build -ldflags "-X diskclean/internal/report.Version=1.2.0" -o .\dist\diskclean.exe ./cmd/diskclean
```

**Test:**
```bash
go test ./...                    # all tests
go test ./internal/cleaner       # single package
go test -run TestRunDryRun       # single test or pattern
```

**Lint:**
```bash
go vet ./...
gofmt -l internal cmd
```

**Run (dry-run recommended for development):**
```bash
./dist/diskclean.exe --dry-run --log-dir ./logs
```

Keep `Winapp2.ini` next to `dist\diskclean.exe` at runtime. The INI file is intentionally ignored by Git and remains an external runtime file.

## Architecture

**Entry point:** `cmd/diskclean/main.go` handles UAC elevation for `--all-users` mode via `internal/elevation`, then delegates to `internal/app.Run`.

**Orchestration:** `internal/app` parses flags, opens the audit log, reads `Winapp2.ini`, discovers user profiles when `--all-users` is set, runs the cleaner, writes summaries, and sends the optional cleanup report. Returns exit code 0 for success, 1 for failures, 2 for invalid arguments.

**Core cleaning logic:** `internal/cleaner` receives parsed Winapp2 sections and options, then:
1. Checks section detection rules (DetectFile, Detect registry paths) to activate sections.
2. Expands environment variables and user-profile-scoped variables (`%UserProfile%`, `%AppData%`, `%LocalAppData%`, `%LocalLowAppData%`, `%Documents%`) via `Expander`. Multi-user mode expands each rule once per discovered user profile.
3. Evaluates FileKey rules with directory wildcards (`*`, `?`), semicolon-separated file patterns, and optional modes (`RECURSE`, `REMOVESELF`).
4. Matches candidates against ExcludeKey rules and deduplicates by case-insensitive cleaned path.
5. Deletes files (or records dry-run candidates) and tracks deleted bytes by directory and user profile.
6. Skips reparse points (junctions, symlinks) at all stages.
7. Validates volume-root rules to reject recursive, self-removing, or match-all patterns on roots.

The cleaner is stateless except for the seen-path deduplication map and the size accumulators. All side effects (deletion, audit recording) pass through injected functions in `Options`.

**Path expansion:** `internal/cleaner/path.go` handles variable substitution, directory wildcards, ProgramFiles(x86) variants, synthetic variables (`%LocalLowAppData%`, `%Documents%`), and per-user profile lookups. `MatchWildcard` uses case-insensitive regex conversion for `*` and `?`.

**Winapp2 parser:** `internal/winapp2` reads INI sections and rules. Malformed FileKey rules are recorded as section Issues and logged as `type=invalid-rule` without halting the section. Registry deletion rules (RegKey) and registry excludes (ExcludeKey=REG) are silently ignored.

**Audit logging:** `internal/audit` writes timestamped structured logs with event type, section, path, reason, size-bytes, and human-readable size. The logger:
- Filters routine skip events (`section-skipped`, `warning-skipped`, `skipped`, `dry-run`) from the on-disk log via `shouldSkip`, but counts still appear in the summary.
- Writes directory and user summaries as Chinese-header blocks (`以下为目录总结`, `以下为用户总结`).
- Enables NTFS compression on the log directory when supported (logs `compression-skipped` on failure).
- Auto-deletes `diskclean-*.log` files older than 15 days, leaving other files untouched.

**Remote reporting:** `internal/report` posts a JSON cleanup report to the configured endpoint after the audit log is written. The payload includes per-user deleted sizes in GB and an overall total, but never the full audit log. A failure is logged as `type=report-skipped` and printed to stderr without changing the exit code. Pass `--report-url=` to disable reporting.

## Patterns

- **Dependency injection for testability:** The cleaner accepts `Remove`, `Audit`, `Registry`, and `Expander` as options. Tests inject fakes and spies.
- **Case-insensitive Windows paths:** Deduplication, matching, and profile detection use `strings.ToLower` and `filepath.Clean`.
- **Reparse point safety:** `isReparsePoint` checks `FILE_ATTRIBUTE_REPARSE_POINT` on Windows. Reparse points are skipped during directory expansion, candidate scanning, and recursive walks.
- **Validation before action:** `ValidateTarget` rejects dangerous root rules before any file-system operation.
- **Deterministic test data:** Use `t.TempDir()`, `testing/fstest.MapFS`, and injected clocks (`time.Now` passed to `audit.Open`).
- **Stateless size formatting:** `cleaner.FormatSize` (1.23 KB / 4.56 MB / 7.89 GB) and `cleaner.FormatSizeGB` (0.00 GB minimum) are pure functions reused across audit, terminal output, and the report payload.

## Windows-Specific Code

- `internal/elevation/elevation_windows.go` — UAC re-launch via ShellExecuteEx with `runas` verb.
- `internal/cleaner/registry_windows.go` — Registry detection via `windows.RegOpenKeyEx`.
- `internal/cleaner/reparse_windows.go` — Reparse point detection via `FILE_ATTRIBUTE_REPARSE_POINT`.
- `internal/report/osinfo_windows.go` — OS name from registry `ProductName`, processor architecture from `GetNativeSystemInfo`.
- `internal/audit/audit.go` — NTFS compression via `FSCTL_SET_COMPRESSION`.

All Windows syscalls use `golang.org/x/sys/windows`.

## Testing Notes

- Most tests use dry-run mode or inject a fake `Remove` function to avoid actual file deletion.
- `app_test.go` uses real temporary directories and validates the full integration including audit log output and remote reporting via `httptest.Server`.
- Test names follow Go convention: `Test<Function><Scenario>` (e.g., `TestRunDryRunRecordsAndTotalsCandidateBytes`).
- Use `t.Logf` to print the audit log path when debugging test failures.

---
> Source: [mchongjs/DiskClean-Cli](https://github.com/mchongjs/DiskClean-Cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
