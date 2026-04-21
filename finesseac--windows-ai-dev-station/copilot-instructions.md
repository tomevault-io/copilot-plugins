## windows-ai-dev-station

> - Use TypeScript strict mode for all new JS/TS code


# Global Development Rules

## Code Standards
- Use TypeScript strict mode for all new JS/TS code
- Python: Use type hints, dataclasses/pydantic, follow PEP 8
- PowerShell: Use approved verbs, comment-based help headers
- Imports: Organized, absolute paths, no unused imports

## Git Practices
- Conventional commits: `feat|fix|docs|refactor|test|chore(scope): message`
- Never commit secrets, API keys, or credentials
- Squash merge feature branches
- Keep commits atomic — one logical change per commit

## Windows-Specific
- Use forward slashes in source code, backslashes only in PowerShell/CMD
- Test WSL2 interop for cross-platform scripts
- Use `$env:USERPROFILE` not `~` in PowerShell
- Use `[System.IO.Path]::Combine()` for cross-platform path construction

## AI Workflow
- Always check for existing implementations before creating new ones
- Preserve existing comments and documentation
- Run linter after code changes
- Log all significant operations for audit trail
- Reference environment variables for secrets, never hardcode

## Error Handling
- Fail fast with clear error messages
- Use structured error types, not string matching
- Log errors with context (timestamp, operation, input)
- Auto-retry transient failures (network, file locks) up to 3 times

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FinesseAC) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
