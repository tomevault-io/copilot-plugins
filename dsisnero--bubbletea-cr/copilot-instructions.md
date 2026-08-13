## bubbletea-cr

> Crystal port of Go's bubbletea TUI framework

# bubbletea.cr

Crystal port of Go's bubbletea TUI framework

## Commands

```bash
make install                 # Install dependencies
make update                  # Update dependencies
make format                  # Check code formatting
make lint                    # Run ameba linter (fix then verify)
make test                    # Run specs
make clean                   # Clean temporary files
make build-examples          # Build example applications
make check-go-port-inventory # Check Go port inventory
make check-go-source-parity  # Check Go source parity
make check-go-test-parity    # Check Go test parity
make parity-specs            # Run example parity specs with local caches
make parity-shell            # Print parity environment setup command
```

## Documentation

| Document | Purpose |
|----------|---------|
| [Architecture](docs/architecture.md) | System design, data flow, package responsibilities |
| [Development](docs/development.md) | Prerequisites, setup, daily workflow |
| [Coding Guidelines](docs/coding-guidelines.md) | Code style, error handling, naming conventions |
| [Testing](docs/testing.md) | Test commands, conventions, patterns |
| [PR Workflow](docs/pr-workflow.md) | Commits, PRs, branch naming, review process |
| [Porting Parity](docs/porting-parity.md) | Upstream source tracking and parity verification |
| [Upgrade Guide v2](docs/upgrade_guide_v2.md) | Migration from v1 to v2 (Crystal-specific) |

## Core Principles

1. Upstream Go code is source of truth
2. Preserve behavior exactly
3. Don't skip functionality
4. Maintain test parity
5. Follow Crystal best practices

## Commits

Format: `<type>(<scope>): <description>`

Types: feat, fix, docs, refactor, test, chore, perf

Examples:

- feat(tea): add mouse event handling
- fix(renderer): correct cursor positioning
- docs(README): update installation instructions

## Crystal Code Gates

```bash
crystal tool format src spec
ameba src spec
crystal spec
rumdl fmt docs/ *.md
```

## External Dependencies

- **Upstream Go repository**: `vendor/bubbletea` submodule must be at correct
  commit
  - Verification: `git submodule status vendor/bubbletea`
  - Update: `git submodule update --init --remote vendor/bubbletea`

## Debugging

When something breaks, check parity with upstream Go code first. Compare
behavior with `vendor/bubbletea` at the same commit.

## Conventions

- Use Crystal idioms without changing Go semantics
- Port test logic exactly from Go tests
- Mark missing functionality as `pending` in specs
- Follow Go test patterns for coverage and edge cases
- Use `./temp` directory for temporary files
- Never commit temporary files to git

---
> Source: [dsisnero/bubbletea.cr](https://github.com/dsisnero/bubbletea.cr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
