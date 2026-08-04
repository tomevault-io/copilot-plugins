## zigbind

> See @README.md for information.

# Zigbind Agent Instructions

## Project Objective

See @README.md for information.

## Tech Stack

### Core
- **Zig 0.15.x**: Primary implementation language, build system
- **Python 3.12+**: Target runtime, using Limited API (Stable ABI)
- **Python C API**: Limited API subset for forward compatibility

### Development Tools
- **uv**: Python package/environment management
- **pytest**: Testing framework
- **ruff**: Python linting and formatting (replaces black, isort, flake8)
- **zig build**: Native build system (no external build tools needed)

## API Philosophy

**IMPORTANT**: This project is in early development. The API is actively being defined and refined.

- **No backward compatibility guarantees**: Breaking changes are expected and acceptable
- **Examples must stay current**: Always update all examples when changing the API
- **Prefer ergonomics over compatibility**: If a better API design is found, implement it immediately
- **Document current API only**: Remove legacy/deprecated APIs entirely rather than maintaining them

## Testing

Tests are organized by feature with their own Zig modules compiled together. All tests include a build step (`test_00_build`) that automatically compiles the Zig modules on first run.

```bash
cd tests

# Run all tests (builds all test modules automatically)
uv run pytest . -v

# Run specific test file
uv run pytest test_functions.py -v      # Basic function types
uv run pytest test_containers.py -v     # List container support
uv run pytest test_memory_leak.py -v    # Memory leak stress tests
```

## Resources

- [Python C API](https://docs.python.org/3/c-api/extension-modules.html)
- [Python C API stability](https://docs.python.org/3/c-api/stable.html)
- [Zig language reference](https://ziglang.org/documentation/master/)
- [Zig standard library](https://ziglang.org/documentation/master/std/)
- [Zig Build System](https://ziglang.org/learn/build-system/)
- [nanobind (inspiration)](https://github.com/wjakob/nanobind)
- [PEP 517 (Build Backend)](https://peps.python.org/pep-0517/)

---

*This file should be updated as the project evolves. Keep it accurate and concise.*

---
> Source: [tudoroancea/zigbind](https://github.com/tudoroancea/zigbind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
