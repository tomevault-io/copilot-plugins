## rcv-app

> > Guidance for AI agents working in this repository. Read this file before making any changes.

# AGENTS.md

> Guidance for AI agents working in this repository. Read this file before making any changes.

## Core Architecture

This project follows the standard Go project layout to ensure clear separation between the transport layer (HTTP/HTMX) and the domain logic (Ranked Choice Voting).

## 1. Project Directory Structure

```
.
├── cmd/
│   └── server/
│       └── main.go           # Entry point: dependency injection & server start
├── internal/                 # Private packages (inaccessible to external modules)
│   ├── voting/               # Domain Logic: RCV Tabulation algorithm
│   ├── models/               # Data Models: Shared structs
│   ├── database/             # Persistence: DB Connection & Repository patterns
│   └── handlers/             # Transport: HTTP/HTMX Handlers (Controllers)
├── ui/                       # Frontend Assets
│   ├── html/                 # Go Templates (.html or .tmpl)
│   └── static/               # Static assets (CSS, JS, HTMX source)
├── db/
│   └── migrations/           # SQL Migration files
├── go.mod                    # Module definition
└── go.sum                    # Dependency checksums
```

## 2. Dependency Management

- **Go Modules**: All internal imports must be prefixed with the module name defined in `go.mod`. Example: `github.com/egp/rcv-app/internal/voting`.
- **Standard Library**: Favor the Go standard library (`net/http`, `html/template`) where possible to minimize external dependencies.

## 3. Development Standards

### Backend Logic

- **Package Isolation**: Business logic (e.g., the `Tabulate` function) must reside in `internal/voting` and remain agnostic of the HTTP layer.
- **Exported Identifiers**: Structs and functions required across package boundaries must be capitalized (e.g., `models.Ballot`, `voting.Tabulate`).
- **Error Handling**: Errors must be treated as values. Handlers should catch domain errors and return appropriate HTML error fragments for HTMX to swap.

### RCV Tabulation & Tiebreaking

The `Tabulate` function in `internal/voting` implements the following elimination logic. Agents modifying this package must preserve this behavior exactly — the test suite verifies each layer independently.

1. **First-choice vote counts**: Each active ballot contributes one vote to its highest-ranked non-eliminated candidate.
2. **Majority check**: If any candidate holds strictly more than 50% of active votes, they win immediately.
3. **Elimination**: The candidate(s) with the fewest votes are identified as tied losers.
4. **Borda tiebreaker**: If multiple candidates are tied at the minimum, compute a Borda score for each tied candidate across all active ballots (1st place = N−1 points, 2nd = N−2, ..., last = 0, where N is the number of active candidates). Eliminate the tied candidate with the lowest Borda score.
5. **Lowest-ID fallback**: If Borda scores are also equal among tied candidates, eliminate the candidate with the lowest ID.

This chain — RCV → Borda → lowest-ID — must be preserved in that order. See `TestTabulate_BordaResolvesToNonLowestID` and `TestTabulate_BordaAlsoTiedFallsBackToLowestID` for concrete examples of each layer being exercised.

### HTMX & Templating

- **Fragment-Driven Design**: Use `html/template` blocks to define components. Handlers should be capable of returning either a full page or a partial fragment based on the presence of the `HX-Request` header.
- **State Management**: Application state is managed server-side. The UI reflects the current state through server-driven hypermedia swaps.
- **Efficiency**: Use `hx-boost` for full-page transitions and `hx-indicator` for long-running calculations (like RCV tabulation rounds).

**Template Layout and Blocks**
Use a base layout template (`ui/html/base.html`) as the foundation for all pages.

- **Base Layout Structure**: The base template defines top-level blocks (`"title"`, `"head"`, `"content"`) that pages can override. Shared components (`"nav"`, `"base_head"`) are defined as reusable templates within the base file.
- **Page Overrides**: Individual page templates (e.g., `home.html`, `vote.html`) only define the blocks they need to customize using `{{define "blockName"}}`.
- **Inclusion and Execution**: Pages are rendered by executing the base template: `template.ExecuteTemplate(w, "base", data)`.
- **Best Practices**:
  - Keep blocks semantic and minimal — `"content"` for main page body, `"head"` for page-specific styles/scripts.
  - Avoid deeply nested blocks. Define sub-components as separate templates (e.g., in `partials.html`) and include via `{{template "componentName" .}}`.
  - For HTMX fragments, check the `HX-Request` header and render the relevant block independently.
  - Validate templates at build time to catch syntax errors early.

### Static Assets

All static files live in `ui/static/`. Do not inline styles or scripts in HTML templates.

**CSS**
- `ui/static/main.css` — global styles shared across all pages. Add shared rules here.
- `ui/static/<page>.css` — page-specific styles (e.g., `home.css`, `poll-created.css`). Create one per page only when styles are not shared.
- Load `main.css` unconditionally in the base layout's `<head>`. Load page-specific stylesheets by overriding the `"head"` block in the relevant page template:
```html
  {{define "head"}}
    <link rel="stylesheet" href="/static/<page>.css">
  {{end}}
```

**JavaScript**
- Place page-specific scripts in `ui/static/<page>.js` (e.g., `poll-created.js`).
- Reference scripts using `defer` to avoid blocking HTML parsing:
```html
<link rel="stylesheet" href="/static/<page>.css">
<script src="/static/poll-created.js" defer></script>  
```
- Load page-specific scripts by overriding the `"head"` block alongside any page-specific CSS.

## 4. Agent Guidelines for Code Changes

### Running Commands

Use these exact shell commands — do not invent alternatives:

| Task                             | Command                                    |
| -------------------------------- | ------------------------------------------ |
| Run all tests                    | `go test ./...`                            |
| Run tests with verbose output    | `go test -v ./...`                         |
| Run tests for a specific package | `go test ./internal/voting/...`            |
| Format code                      | `gofmt -s -w .`                            |
| Lint                             | `staticcheck ./...`                        |
| Build the binary                 | `go build -o rcv-app ./cmd/server/main.go` |
| Tidy dependencies                | `go mod tidy`                              |

### Editing Files

Use a string-replacement approach with exact old and new strings. Always include 3–5 lines of surrounding context in the old string to guarantee uniqueness. Never edit a file without reading it first in the same session — stale context causes mismatched replacements.

### File Paths and Imports

Always use absolute paths when referencing files. For Go imports, use the full module path (e.g., `github.com/egp/rcv-app/internal/voting`). Do not invent paths — verify with file-reading tools if uncertain.

### Code Patterns

- **Handlers** in `internal/handlers/` must check the `HX-Request` header:
  ```go
  if r.Header.Get("HX-Request") != "" {
      // render fragment
  } else {
      // render full page
  }
  ```
- **Templates**: Use `{{define "blockName"}}` for page-specific blocks; execute via `template.ExecuteTemplate(w, "base", data)`.
- **Errors**: Return errors as values; in handlers, render error fragments like `{{template "error" .}}`.

### Security

- Implement CSRF protection for all non-GET requests that use HTMX.
- Do not modify production configuration files without explicit confirmation from the user.
- Do not read, log, or transmit secrets (environment variables, key files, credentials).

## 5. Database Migrations

The `db/migrations/` directory holds SQL migration files. No migration tool has been chosen yet. Until one is adopted, follow these rules:

- **Do not create or modify migration files** without explicit instruction from the user.
- **Do not run destructive SQL** (`DROP`, `DELETE`, `TRUNCATE`) against any database without confirmation.
- When a migration tool is selected, update this section with: the tool name, install command, how to create a new migration, how to run migrations up/down, and how to check current migration status.

## 6. Implementation Checklist

Before marking any task complete, verify all of the following:

- [ ] **Module imports**: All local package imports follow `github.com/egp/rcv-app/internal/<package>`. No invented or shortened paths.
- [ ] **Type safety**: Domain-specific ID types (e.g., `type CandidateID int`) are used consistently across the voting logic — do not substitute bare `int` where a named type is expected.
- [ ] **RCV tiebreaker chain**: Any change to `internal/voting` preserves the Borda → lowest-ID fallback order. Run `go test ./internal/voting/...` and confirm all tiebreaker tests pass.
- [ ] **Template embedding**: Use `//go:embed` to bundle the `ui/` directory into the binary for production builds.
- [ ] **CSRF**: All non-GET HTMX handlers have CSRF protection in place.
- [ ] **No lint errors**: `staticcheck ./...` is clean.
- [ ] **Formatted**: `gofmt -s -w .` produces no diff.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emiliano-ponce)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/emiliano-ponce)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
