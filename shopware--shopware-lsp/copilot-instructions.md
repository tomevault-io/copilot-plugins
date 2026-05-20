## shopware-lsp

> Shopware LSP is a Language Server Protocol implementation for Shopware and Symfony development. It provides IDE features (completion, go-to-definition, hover, diagnostics) for PHP, Twig, XML, and YAML files.

# Project Overview

Shopware LSP is a Language Server Protocol implementation for Shopware and Symfony development. It provides IDE features (completion, go-to-definition, hover, diagnostics) for PHP, Twig, XML, and YAML files.

**Tech Stack:** Go backend with tree-sitter parsing, BBolt embedded database for indexes, TypeScript VSCode extension.

## Build & Test Commands

```bash
# Build
go build                          # Build LSP server binary
go build ./...                    # Build all packages

# Test
go test ./...                     # Run all tests
go test -race ./internal/...      # Race detection (used in CI)
go test ./internal/php/... -v     # Test specific package
go test -run TestFeatureIndexer   # Run specific test

# Lint
golangci-lint run                 # Lint check (run before committing)

# VSCode extension
cd vscode-extension
npm install && npm run compile    # Build extension
npm run check-types               # Type check only
```

## Architecture

### Entry Point
`main.go` initializes the LSP server, registers all indexers and providers, then starts on stdin/stdout (JSON-RPC).

### Key Packages (`internal/`)

| Package | Purpose |
|---------|---------|
| `lsp/` | LSP protocol, server.go is the main handler |
| `lsp/completion/` | 7 completion providers (services, routes, twig, snippets, features, system config, theme) |
| `lsp/definition/` | 7 go-to-definition providers (same domains) |
| `indexer/` | FileScanner for file watching, DataIndexer for BBolt persistence |
| `symfony/` | Service container and route indexing from XML/YAML/PHP |
| `php/` | PHP class/method indexing, type inference, alias resolution |
| `twig/` | Template indexing, block tracking, extends/include parsing |
| `snippet/` | Translation key indexing from JSON files |
| `feature/` | Feature flag indexing from YAML |
| `theme/` | Theme config and icon indexing |
| `extension/` | Shopware extension metadata from composer.json |

### Provider Pattern
All LSP features use a provider interface pattern. Multiple providers can handle the same feature type, routed by document language/context.

### Indexing Flow
1. `FileScanner` detects file changes (fsnotify)
2. Files parsed with tree-sitter based on type
3. Each registered indexer processes AST nodes
4. Data persisted to BBolt databases in cache directory

## Testing Patterns

Tests use `testify/assert` and `testify/require`. Common pattern:
```go
func TestSomething(t *testing.T) {
    tempDir := t.TempDir()
    indexer, err := NewIndexer(tempDir)
    require.NoError(t, err)
    defer indexer.Close()
    // ... test logic
}
```

## Commit Guidelines

Use Conventional Commits: `type(scope): summary`
- `fix:` bug fixes
- `feat:` new features
- `build:` build system changes
- `docs:` documentation

## Debug Tool

```bash
go run cmd/debug_ast/main.go path/to/file.php  # Visualize PHP AST
```

---
> Source: [shopware/shopware-lsp](https://github.com/shopware/shopware-lsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
