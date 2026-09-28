## criticalaisurveys

> **TODO.md is the source of truth** for determining the current state of the project and what tasks need to be completed. Always check TODO.md first when:

# Claude Assistant Instructions

## Project State Management

**TODO.md is the source of truth** for determining the current state of the project and what tasks need to be completed. Always check TODO.md first when:

- Understanding what has been completed
- Determining next steps
- Planning new features or fixes
- Assessing project progress

## Git Commit Guidelines

When writing commit messages to GitHub:

- **Never mention Claude or AI assistance** in commit messages or commit body
- **Do not include AI attribution footer** (no "Generated with Claude Code" or "Co-Authored-By: Claude")
- Write commit messages as if they were written by a human developer
- Focus on the technical changes and business value
- Use conventional commit format when applicable
- Keep commits clean and professional

### Examples:

❌ **Don't do this:**
```
Add user authentication with Claude's help

🤖 Generated with [Claude Code](https://claude.ai/code)
Co-Authored-By: Claude <noreply@anthropic.com>
```

❌ **Also don't do this:**
```
Fix bug identified by AI assistant
```

✅ **Do this instead:**
```
Add user authentication with JWT tokens

- Implement JWT token generation and validation
- Add authentication middleware for protected routes
- Update user model with token refresh capabilities
```

✅ **Or this:**
```
Fix null pointer exception in checkout flow

Resolve NPE that occurred when processing orders without shipping addresses
```

## Development Workflow

1. Check TODO.md for current project state
2. Update TODO.md as work progresses
3. Write human-style commit messages
4. Keep documentation updated alongside code changes

---
> Source: [FernandoTN/CriticalAiSurveys](https://github.com/FernandoTN/CriticalAiSurveys) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
