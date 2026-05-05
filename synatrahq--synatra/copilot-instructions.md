## synatra

> - Simplicity outweighs sophistication

## Principles
- Simplicity outweighs sophistication
- Code must justify its existence
- Minimize to maximize impact

## Coding
- Consolidate logic in single functions unless reuse is clear
- Avoid excessive destructuring, else branches, and try/catch
- Use guard clauses and early returns
- Favor concise variable names
- Use double quotes
- All user-facing text must be English
- Tailwind colors must use semantic CSS vars in @layer base

## Structure
- Match existing patterns in the codebase
- Modify existing files before creating new ones
- Omit comments unless explicitly requested
- Use kebab-case naming; avoid generic names; names describe purpose; singular folder names

## Errors & Security
- Fail fast with clear messages; prefer returning errors as values
- Log errors only when actionable
- Never log or commit secrets
- Validate all inputs; use parameterized queries to prevent injection

## Workflow
- Check Makefile for available commands; propose additions after verification
- Run typecheck and tests for all functional changes
- Keep functional and cosmetic changes in separate commits
- Commit only when requested; keep commits focused and atomic
- Review package.json before adding deps; prefer stdlib; minimize dependencies; verify latest versions before install

## Testing
- Write tests alongside functional changes
- Ensure tests cover edge cases and error paths
- Maintain test isolation and reproducibility

## Docs
- Create or update documentation only when requested or when behavior changes
- Keep documentation synchronized with code changes

---
> Source: [synatrahq/synatra](https://github.com/synatrahq/synatra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
