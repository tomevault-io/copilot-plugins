## magus

> - **NEVER make commits without explicit user approval**

# Cursor AI Rules for MAGUS Project

## Git Workflow
- **NEVER make commits without explicit user approval**
- **NEVER run destructive git commands (push --force, hard reset) without explicit approval**
- **NEVER skip hooks (--no-verify, --no-gpg-sign) unless explicitly requested**
- Always propose git commands for user review before executing
- Use clear, conventional commit messages when approved

## Code Style
- Follow PEP 8 for Python code
- Use Black formatter for Python
- Use ESLint + Prettier for JavaScript/TypeScript
- Use meaningful variable and function names
- Add docstrings to all functions and classes

## Development Workflow
- Create feature branches for new work
- Keep commits atomic and focused
- Test changes before committing
- Update documentation when changing functionality

## Django Specific
- Always create migrations after model changes
- Use Django ORM, avoid raw SQL
- Follow Django REST Framework best practices for serializers and viewsets
- Keep views thin, business logic in models/services

## React/TypeScript Specific
- Use TypeScript strict mode
- Prefer functional components with hooks
- Keep components small and focused
- Use TanStack Query for server state
- Use Zustand for client state

## Documentation
- **NEVER create documentation files (.md) unless explicitly requested by the user**
- Update README when adding features (only if explicitly asked)
- Document API changes in OpenAPI schema
- Keep PLAN.md and SPRINTS.md up to date (only when explicitly working on those)
- Add inline comments for complex logic
- Do not create helper docs, guides, or summary files proactively

## Testing
- Write tests for new features
- Maintain 80%+ coverage
- Test edge cases and error conditions

## Security
- Never commit secrets or API keys
- Use environment variables for configuration
- Validate all user inputs
- Follow OWASP best practices

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dinjou) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
