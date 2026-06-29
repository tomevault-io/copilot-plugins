## fingergo

> - Follow idiomatic Go and vanilla JavaScript best practices.

# FingerGo Agents Context

## Core Principles

- Follow idiomatic Go and vanilla JavaScript best practices.
- Keep changes minimal: the fewer lines of code, the better.
- Prefer existing logic, utilities, modules, and patterns over new abstractions.
- Abstract only when it removes real duplication or clarifies an existing boundary.
- Preserve existing style, formatting, comments, and public behavior.
- Use SOLID, SRP, DRY, and YAGNI pragmatically.
- Do not introduce frontend frameworks or bundlers; the GUI is intentionally vanilla.

## Documentation Map

- [README](README.md) - product overview, install notes, high-level tech stack.
- [Architecture](docs/tech-design/architecture.md) - system design, module boundaries, data flow.
- [About FingerGo](docs/tech-design/about-fingergo.md) - project background and goals.
- [Text Management](docs/tech-design/text-management.md) - text library and category design.
- [Settings](docs/tech-design/settings.md) - settings persistence and behavior.
- [Statistics Analytics](docs/tech-design/statistics-analytics.md) - session stats and analytics.
- [Keyboard](docs/tech-design/keyboard.md) - visual keyboard and layout behavior.
- [Typing Modes](docs/tech-design/typing-modes.md) - typing mode design.
- [Contributing](docs/CONTRIBUTING.md) - local setup, CI expectations, contribution notes.
- [Attribution](docs/ATTRIBUTION.md) - third-party credits.

## Command Reference

### Build & Dependencies

```bash
# Install all dependencies (Go + JS)
make deps

# Build the production binary
make build

# Start Wails development mode with hot reload
make dev

# Run the application through Wails
make run

# Generate Wails bindings
make generate

# Remove build artifacts and generated frontend dependencies
make clean
```

### Quality & Tests

```bash
# Format Go and JS
make fmt

# Lint Go and JS
make lint

# Run Go and JS tests
make test

# Check license headers
make license

# Run full test/build verification script
make test-build
```

### Direct Go Commands

```bash
# Format Go
gofmt -s -w .
goimports -w .

# Lint and vet
golangci-lint run
go vet ./...

# Run all Go tests
go test ./...

# Run race detector like CI
go test -race ./...
```

### Direct JavaScript Commands

```bash
# Install dependencies
npm install

# Lint GUI code
npm run lint

# Fix lint issues where safe
npm run lint:fix

# Format GUI assets
npm run format

# Check formatting
npm run format:check

# Run Node test suite
npm test
```

### Packaging & Release Helpers

```bash
# Generate platform icons
make icons

# Linux build used by CI
wails build -platform linux/amd64 -tags webkit2_41

# Generic Wails build
wails build
```

## Project Structure

- `main.go` - Wails desktop entry point, app window options, embedded GUI assets.
- `app/` - application layer and Wails-bound API methods.
- `internal/domain/` - domain types for texts, categories, sessions, settings, languages.
- `internal/storage/` - JSON persistence, embedded defaults, validation, data paths.
- `internal/storage/embedded/` - default text library data embedded into the binary.
- `gui/src/index.html` - static HTML entry point.
- `gui/src/js/` - vanilla ES modules for app orchestration and UI behavior.
- `gui/src/js/modals/` - modal components.
- `gui/src/js/layouts/` - keyboard layout definitions.
- `gui/src/js/__tests__/` - Node test files for frontend utilities and events.
- `gui/src/styles/` - CSS split by layout, modals, keyboard, library, themes.
- `flatpak/` - Flatpak manifest and generated Go module source metadata.
- `build/` - platform build resources and Wails output.
- `.github/workflows/` - CI, Flatpak, and release workflows.

---
> Source: [AshBuk/FingerGo](https://github.com/AshBuk/FingerGo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
