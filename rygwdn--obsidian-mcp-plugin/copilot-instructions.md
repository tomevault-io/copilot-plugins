## commit-message-format

> Instructions for writing commit messages

# Commit Message Format

All commits to this codebase should follow the conventional commit format:

## Format
```
<type>: <concise description>

[optional body with details]
```

## Types
- `feat`: New features or significant enhancements
- `fix`: Bug fixes
- `docs`: Documentation changes
- `style`: Code style/formatting changes (no functional changes)
- `refactor`: Code changes that neither fix bugs nor add features
- `perf`: Performance improvements
- `test`: Adding or correcting tests
- `chore`: Maintenance tasks, dependencies, etc.

## Guidelines
- Keep the first line under 72 characters
- Use present tense ("add feature" not "added feature")
- Be specific about what changed
- Reference issue numbers in the body when applicable
- Do not mention Claude or other AI assistants in commit messages

## Examples
```
feat: add daily note resources to MCP
fix: resolve issue with file path normalization
docs: update README with new tools documentation
refactor: consolidate resource tools and tests

---
> Source: [rygwdn/obsidian-mcp-plugin](https://github.com/rygwdn/obsidian-mcp-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
