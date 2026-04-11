## backgroundagent

> This file provides project-specific context and instructions for the Gemini CLI agent.

# Gemini CLI Configuration

This file provides project-specific context and instructions for the Gemini CLI agent.

## Project Overview

This is a self-driving code base powered by GitHub Actions and Gemini CLI. The agent helps with:
- Issue triage and classification
- Code review
- Documentation maintenance
- Bug diagnosis
- Feature implementation

## Code Style Guidelines

### General Principles
- Write clean, readable code with meaningful names
- Follow existing patterns in the codebase
- Keep functions small and focused (single responsibility)
- Add appropriate error handling
- Write tests for new functionality

### JavaScript/TypeScript
- Use `const` and `let`, avoid `var`
- Use arrow functions where appropriate
- Use meaningful variable names
- Add JSDoc comments for complex functions
- Use async/await over raw promises
- Use template literals for string interpolation

### Python
- Follow PEP 8 style guide
- Use type hints where beneficial
- Use list/dict comprehensions where appropriate
- Add docstrings to modules and functions
- Use `logging` instead of `print` for important output

### Go
- Follow Go code review comments
- Use meaningful package names
- Error handling: `if err != nil { return err }`
- Use context for cancellation
- Write table-driven tests

### Documentation
- Update README when adding features
- Add inline comments for complex logic
- Document public APIs
- Include usage examples for new features

## Commit Message Format

```
type(scope): description

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

## Pull Request Guidelines

- Reference the issue number in the PR title
- Describe what changes were made
- Include screenshots for UI changes
- Ensure all tests pass
- Update documentation as needed

## Testing Requirements

- Unit tests for new functions/methods
- Integration tests for new features
- Test edge cases and error conditions
- Aim for >80% coverage on new code

## Security Considerations

- Never commit secrets or API keys
- Sanitize user input
- Use parameterized queries
- Validate all inputs
- Follow OWASP security guidelines

## File Organization

```
src/          - Source code
tests/        - Test files
docs/         - Documentation
scripts/      - Build/utility scripts
config/       - Configuration files
```

## CI/CD Pipeline

1. **Lint** - Code style checks
2. **Test** - Run test suite
3. **Build** - Compile/build artifacts
4. **Deploy** - Deploy to environment

## Getting Help

- Check existing issues and PRs
- Review documentation
- Ask in discussions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RahRha-v3-2)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RahRha-v3-2)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
