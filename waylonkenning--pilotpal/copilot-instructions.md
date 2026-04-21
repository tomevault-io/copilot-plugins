## pilotpal

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Workflow

Follow this exact workflow for all feature development:

1. **Update todo list** - Use TodoWrite to plan and track all tasks
2. **Create e2e tests** - Write end-to-end tests for the new feature before implementation
3. **Code the feature** - Implement the feature according to the tests
4. **Run local validation** - Execute `./scripts/pre-commit-check.sh` to verify:
   - TypeScript type checking (catches build errors)
   - ESLint code quality checks  
   - Build verification (same as Vercel)
   - Unit tests (if any exist)
5. **Git commit** - Commit changes with descriptive message
6. **Git push** - Push to remote repository
7. **Wait for Vercel deployment** - Allow deployment to complete
8. **Test deployed version** - Manually verify feature works in production
9. **Update README** - Document the new feature once fully tested and deployed

This workflow ensures quality through local validation, eliminating CI/CD complexity while maintaining code standards.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/waylonkenning) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
