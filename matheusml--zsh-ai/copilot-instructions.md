## zsh-ai

> This is a lightweight ZSH plugin that provides AI-powered command suggestions in the terminal. The codebase is pure shell script with zero runtime dependencies.

# Project Development Guide for zsh-ai

## Overview
This is a lightweight ZSH plugin that provides AI-powered command suggestions in the terminal. The codebase is pure shell script with zero runtime dependencies.

## Testing

### Running Tests
```bash
# Run all tests
./run-tests.zsh

# Run specific test directory
./run-tests.zsh tests/providers

# Run single test file
zsh tests/config.test.zsh
```

### Test Structure
- Tests are located in `tests/` directory
- Provider-specific tests in `tests/providers/`
- Test helper utilities in `tests/test_helper.zsh`
- Custom ZSH-based testing framework with mocking support

### Writing Tests
- Follow existing test patterns in test files
- Use `test_helper.zsh` functions for assertions and mocking
- Each test should have proper setup/teardown
- Mock external commands (curl, jq) rather than making real API calls

## Development Workflow

### Before Committing
1. Run all tests: `./run-tests.zsh`
2. Verify shellcheck passes (runs automatically in CI)
3. Test your changes manually with different providers
4. Ensure no API keys or sensitive data are committed

### Code Style
- Follow existing ZSH scripting patterns
- Use meaningful variable names with proper scoping
- Add error handling for all external commands
- Keep functions small and focused
- Maintain backwards compatibility with older ZSH versions

### Provider Implementation
When adding/modifying AI providers:
1. Implement provider in main script
2. Add comprehensive tests in `tests/providers/`
3. Update configuration documentation
4. Test with actual API calls (use your own keys)

### Common Tasks
- **Adding a new provider**: Copy existing provider pattern, add tests
- **Modifying context detection**: Update context function and add tests in `context.test.zsh`
- **Changing command parsing**: Update widget logic and test in `widget.test.zsh`

### Important Notes
- This is a shell-only project - no Node.js, Python, or other runtimes
- Keep the codebase lightweight (currently <5KB)
- Preserve zero-dependency philosophy (jq is optional)
- Always test with both `#` comment syntax and direct `zsh-ai` command
- CI runs on every PR - ensure tests pass before merging

---
> Source: [matheusml/zsh-ai](https://github.com/matheusml/zsh-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
