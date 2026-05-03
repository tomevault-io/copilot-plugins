## project-launcher

> macOS developer dashboard — discover, organize, and launch projects. Flutter + Rust monorepo managed by Melos.

# Project Launcher

macOS developer dashboard — discover, organize, and launch projects. Flutter + Rust monorepo managed by Melos.

## Quick Reference

| Command | What it does |
|---------|-------------|
| `make dev` | Debug build + run |
| `make build` | Release build (Rust + Flutter) |
| `make test` | Run all tests (Rust + Flutter) |
| `make analyze` | Static analysis (all packages via Melos) |
| `make install` | Build + install to /Applications + codesign |
| `make bootstrap` | Install Melos + resolve all packages |
| `make clean` | Clean all build artifacts |

## Architecture

- **lib/** — Flutter app (screens, services, widgets)
- **rust/** — Native Rust core (git ops, health scoring, stats) via FFI
- **cli/** — Rust CLI tool (`plauncher`)
- **packages/** — Extracted Dart packages (see per-package AGENTS.md):
  - `launcher_theme` — Design system (colors, typography, spacing, skins)
  - `launcher_kit` — UI component library (layout, forms, elements)
  - `launcher_models` — Data models (pure Dart, no Flutter dependency)
  - `launcher_native` — FFI bindings to Rust core + logging
- **vscode-extension/** — VS Code sidebar extension (TypeScript)
- **raycast-extension/** — Raycast commands (TypeScript)
- **badge-service/** — Vercel serverless badge generator (Node.js)

### Dependency Graph

```
launcher_theme → launcher_kit → project_launcher (root app)
launcher_models → project_launcher
launcher_native → project_launcher
rust/libproject_launcher_core.dylib → launcher_native (via FFI)
```

## Code Patterns

- State management: StatefulWidget + service singletons
- Theme: `launcher_theme` package with `SkinProvider` for switchable skins
- Native calls: Dart FFI via `launcher_native` package
- Screens: `lib/screens/<name>_screen.dart`
- Services: `lib/services/<name>_service.dart`
- Widgets by feature: `lib/widgets/<feature>/`

## Testing

| Scope | Command |
|-------|---------|
| Everything | `make test` |
| Flutter app only | `flutter test` |
| All packages (Melos) | `melos run test` |
| Rust core | `cd rust && cargo test` |
| Rust CLI | `cd cli && cargo test` |
| Static analysis | `make analyze` |

## Release

- Patch: `make release-patch`
- Minor: `make release-minor`
- Major: `make release-major`
- Dry run: `make release-dry`
- Full script: `./scripts/release.sh <patch|minor|major> [--dry-run]`

---
> Source: [viveky259259/project-launcher](https://github.com/viveky259259/project-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
