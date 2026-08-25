## indentr

> Indentr is a multi-user, tree-structured note-taking desktop app built with C#/.NET 10 and Avalonia. Notes are stored as Markdown in PostgreSQL. The tree structure is derived from in-note links (`[title](note:UUID)`), not a stored hierarchy. Optional remote sync supports multi-machine/multi-user collaboration.

# Indentr — Claude Code Briefing

## What This Is

Indentr is a multi-user, tree-structured note-taking desktop app built with C#/.NET 10 and Avalonia. Notes are stored as Markdown in PostgreSQL. The tree structure is derived from in-note links (`[title](note:UUID)`), not a stored hierarchy. Optional remote sync supports multi-machine/multi-user collaboration.

See `DESIGN.md` for full architecture documentation. See `README.md` for user-facing docs.

## Build & Run

**Prerequisites:** .NET 10 SDK, PostgreSQL 14+

```bash
# Run the app
dotnet run --project Indentr.UI

# Build the solution
dotnet build Indentr.sln

# Run tests
dotnet test Indentr.Tests

# Start local PostgreSQL (Docker/Podman)
cp .env.example .env
podman-compose up -d
podman exec -it indentr-db psql -U postgres -c "CREATE DATABASE indentr;"
```

On first launch, create a profile with your database connection details. Schema migrations run automatically.

## Project Structure

```
Indentr.Core/    — Domain models and interfaces (no external dependencies)
Indentr.Data/    — PostgreSQL repositories and sync service (Npgsql only)
Indentr.UI/      — Avalonia desktop UI, MVVM viewmodels, controls
Indentr.Tests/   — xUnit + Moq tests
```

Dependency direction: `UI → Data → Core`. Core has zero external dependencies.

## Key Architecture Facts

- **No ORM.** All database access uses raw parameterized SQL via Npgsql. Do not introduce an ORM.
- **Link-derived tree.** Parent-child relationships are derived from `[text](note:UUID)` links in note content, not stored directly. `SyncParentLinksAsync()` maintains these automatically.
- **Conflict detection via SHA-256.** On save, if `current_hash ≠ expected_hash`, a `[CONFLICT]` sibling note is created instead of overwriting. Preserve this behavior.
- **Static service registry.** `App.Notes`, `App.Users`, etc. are static properties set at startup. This is intentional — do not refactor to constructor injection without good reason.
- **Window registry.** `NotesWindow` and `KanbanWindow` maintain `_openWindows` lists for bulk save/reload/close. Keep new windows consistent with this pattern.
- **Offline-first sync.** Local DB is primary; remote sync is best-effort. Sync runs every 10 minutes or on `Shift+Ctrl+S`.

## Coding Conventions

- **C# 10+ style:** Records, nullable reference types (`#nullable enable`), implicit usings, async/await throughout.
- **Naming:** PascalCase for public members; `_camelCase` for private fields.
- **Async-first:** All I/O must be async. Never block on async code with `.Result` or `.Wait()`.
- **Parameterized SQL only.** Never concatenate user input into SQL strings.
- **Privacy is enforced at query level.** Private notes are filtered in SQL (`is_private = false OR owner_id = @userId`). Do not bypass this in new queries.
- **Regex:** Use `RegexOptions.Compiled` for patterns that are reused.

## Things to Be Careful About

- The app supports multiple profiles (databases). Avoid hardcoding connection strings or assuming a single DB.
- Migrations live in `Indentr.Data/Migrations/` as embedded SQL files and run via `DatabaseMigrator`. Add new migrations there; don't alter existing ones.
- Attachments are stored as PostgreSQL large objects. Don't change the storage strategy without reading the design doc first.
- The sync protocol is complex (push/pull phases, clock skew handling, FK ordering). Read `DESIGN.md` § Sync before modifying `SyncService.cs`.
- The app runs on Linux, macOS, and Windows. Avoid platform-specific APIs or path separators.

---
> Source: [scottteague/indentr](https://github.com/scottteague/indentr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
