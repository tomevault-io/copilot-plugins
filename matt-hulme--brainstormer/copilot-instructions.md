## brainstormer

> description: Shared development practices applicable across frontend and backend

---
description: Shared development practices applicable across frontend and backend
globs: ["frontend/src/**/*.{ts,tsx}", "backend/app/**/*.py"]
alwaysApply: true
---
# Shared Development Guide

## Project Structure
Key directories:
- [frontend](mdc:frontend) - React frontend application
- [backend](mdc:backend) - FastAPI backend application
- [.cursor](mdc:.cursor) - Project configuration and rules

## Import Rules
1. No wildcard imports (export * or import *)
2. Import directly from source files
3. Use explicit, named imports
4. Keep imports organized and grouped, but without spaces between them
5. Follow language-specific import conventions
6. Avoid re-exporting from index files
7. Use absolute imports when possible

## File Naming
1. Use descriptive, meaningful names
2. Avoid abbreviations unless widely known
3. Keep file names concise but clear
4. Use appropriate case for your language
5. Follow language-specific conventions

## Environment Configuration
1. Use `.env` for local development
2. Use `.env.example` as a template
3. Never commit `.env` files
4. Document all required environment variables
5. Use environment variables for secrets

## Git Practices
1. Write clear, descriptive commit messages
2. Keep commits focused and atomic
3. Use present tense in commit messages
4. Reference issues when applicable
5. Use feature branches for new features

## Security Guidelines
1. Never commit sensitive data
2. Use environment variables for secrets
3. Follow security best practices
4. Keep dependencies updated
5. Use HTTPS in production

## Error Handling
1. Handle errors gracefully
2. Log errors appropriately
3. Provide meaningful error messages
4. Implement proper error recovery
5. Use consistent error handling patterns

## Performance
1. Optimize database queries
2. Implement proper caching
3. Monitor performance
4. Profile code when needed
5. Follow performance best practices

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Matt-Hulme) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
