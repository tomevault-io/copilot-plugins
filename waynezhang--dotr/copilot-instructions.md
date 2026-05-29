## dotr

> zig build           # Build the project

# DOTR Development Guide

## Build Commands
```bash
zig build           # Build the project
zig build -Doptimize=ReleaseFast  # Release build
zig build test      # Run all tests
zig test src/dotfile/action/link.zig  # Run tests in specific file
```

## Code Style
- **Imports**: std first, then external, then internal modules
- **Formatting**: 4-space indentation, trailing commas in multi-line lists
- **Naming**: `snake_case` for variables/functions, `camelCase` for struct fields
- **Types**: Explicit types for parameters and returns, structs for domain objects
- **Error Handling**: Custom errors with `error.ErrorName`, functions return `anyerror!void`
- **Memory**: Explicit allocator pattern, use `defer` for cleanup
- **Testing**: Tests alongside code using `test "name"` syntax, comprehensive coverage

---
> Source: [waynezhang/dotr](https://github.com/waynezhang/dotr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
