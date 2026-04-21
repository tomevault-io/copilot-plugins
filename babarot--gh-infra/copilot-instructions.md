## gh-infra

> | `cmd` | Cobra CLI command definitions and flag handling |

# CLAUDE.md

## Package Responsibilities

| Package | Role |
|---|---|
| `cmd` | Cobra CLI command definitions and flag handling |
| `internal/infra` | Orchestrator — drives plan, apply, import workflows |
| `internal/fileset` | File-set resolution, templating, patching |
| `internal/importer` | Import logic: match existing repos, reverse-template, write YAML |
| `internal/repository` | Per-repository state, diff, plan, apply, export |
| `internal/manifest` | YAML manifest parsing, validation, type definitions, source expansion, and GitHub identity resolution |
| `internal/gh` | GitHub CLI (`gh`) runner abstraction |
| `internal/ui` | Terminal UI: printer, diff viewer, progress, confirm |
| `internal/parallel` | Generic concurrent map utility |
| `internal/logger` | slog-based structured logger |
| `internal/yamledit` | In-place YAML node editor (preserves formatting) |

## Package Dependency Rules

The internal packages form a clean DAG (no circular dependencies). Respect the following layering:

```
Layer 0 (leaves):  logger, parallel, yamledit   — no internal imports allowed
Layer 1:           gh (→ logger), ui (→ logger)
Layer 2:           manifest (→ gh)
Layer 3:           fileset (→ manifest, gh, parallel), repository (→ manifest, gh, logger, parallel)
Layer 4:           importer (→ fileset, repository, manifest, gh, yamledit)
Layer 5:           infra (→ orchestrator, may import all above + ui)
Layer 6:           cmd (→ infra, ui, manifest, gh, logger, fileset)
```

Key constraints:
- `internal/ui` must NOT depend on domain packages (manifest, repository, infra, etc.) — it is a pure presentation layer
- **Domain packages (repository, importer, fileset, manifest) must NOT depend on `internal/ui` in production code** — the dependency DAG is strictly top-down. When domain packages need progress/diagnostic callbacks, they define their own interfaces and receive implementations from `infra` (Layer 5). Do NOT "unify" these interfaces into `ui` — that would leak presentation concerns into the domain layer. (Test files (`*_test.go`) may import `ui` for test helpers such as formatting utilities; this does not affect the production dependency graph.)
- `internal/manifest` must NOT depend on upper layers — it is the manifest/schema layer and also owns manifest-adjacent resolution logic (for example source expansion and GitHub identity resolution)
- `internal/infra` is the orchestrator; other internal packages must NOT depend on it (except cmd)
- Leaf packages (logger, parallel, yamledit) must NOT import any internal package
- `cmd` should access workflow/domain logic through `infra` where practical. Small direct imports of lower layers are acceptable for thin CLI concerns such as validation/path resolution or shared CLI-only types (for example `manifest` in `validate`, `fileset` for diff-entry helpers).

---
> Source: [babarot/gh-infra](https://github.com/babarot/gh-infra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
