## doodlify

> - **Language**: Python 3.x


# Doodlify - Windsurf Rules

## Tech Stack
- **Language**: Python 3.x
- **CLI Framework**: argparse (stdlib) or click
- **Type hints**: Use Python type annotations
- **Testing**: pytest
- **Packaging**: pyproject.toml with setuptools

## Architecture Principles
- CLI-first design - single entry point
- Fail fast with clear error messages
- Use proper exit codes for CI/CD (0 = success, non-zero = failure)
- JSON output for machine parsing when needed
- Human-readable output by default
 - All generated artifacts MUST be written inside the local workspace clone at `.doodlify-workspace/<repo>/`.
   - The lock filename is derived from the config passed (e.g., `config.json` → `config-lock.json`, `event.manifest.json` → `event.manifest-lock.json`) and saved at `.doodlify-workspace/<repo>/<derived-lock-name>`.
   - Any other generated content (e.g., temp analysis caches, backups created during processing) also resides under the same directory.

## Code Style

### Python
- Use type hints for all functions
- Follow PEP 8 conventions
- Prefer composition over inheritance
- Keep functions small and focused
- Use pathlib for file operations
- Handle exceptions explicitly

### CLI Design
- Clear command structure: `doodlify <command> [options]`
- Provide `--help` for all commands
- Use `--verbose` / `--quiet` flags for output control
- Support `--output json` for CI/CD integration
- Validate inputs early with clear error messages

### Documentation
- Docs files are located in the root README.md and the md files in /docs
- Every important architecture change, including additions and deletions should be updated in the documentation
- The documentation should avoid duplications and long unnecessary explanations
- When multiple content related to the documentation starts growing to around more than a 3rd of the overall information in that md file, then that specialized part of the documentation should have its own .md file.

### Exit Codes
- `0` - Success
- `1` - General error
- `2` - Invalid arguments
- `3` - File/resource not found
- `4` - Validation failure

## CI/CD Requirements
- Must be installable via pip
- Support environment variables for configuration
- Idempotent operations where possible
- No interactive prompts (use flags/env vars)
- Deterministic output for given inputs
- Fast execution (< 5s for typical operations)

## Error Handling
- Log errors to stderr
- Log normal output to stdout
- Use proper logging levels (DEBUG, INFO, WARNING, ERROR)
- Include actionable error messages
- Never swallow exceptions silently

## File Structure
```
doodlify/
├── pyproject.toml
├── README.md
├── src/
│   └── doodlify/
│       ├── __init__.py
│       ├── cli.py          # CLI entry point
│       ├── commands/       # Command implementations
│       └── utils/          # Helper functions
└── tests/
    └── test_*.py
```

## Testing
- Unit tests for all core functions
- Integration tests for CLI commands
- Use pytest fixtures
- Mock external dependencies
- Test exit codes explicitly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EliuX)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EliuX)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
