## conduit

> Act as a senior software engineer. Write clean, minimal, production-ready code. No over-engineering.

# Claude Code Guidelines

## Role
Act as a senior software engineer. Write clean, minimal, production-ready code. No over-engineering.

## Git Practices
- Commit after each completed feature or fix, not mid-work
- Keep commit messages short and descriptive (e.g. `fix mobile metro config`, `add auth endpoint`)
- Do NOT include `Co-Authored-By` in commit messages
- Push only after changes are validated and tested
- Always check `.gitignore` before committing — never commit build artifacts, `node_modules`, `.next`, `.expo`, `dist`, `.env`, or OS files
- Pull before pushing to avoid conflicts

## Testing
- Run all relevant tests before pushing
- If no test suite exists, manually verify the change works
- Check for regressions in related functionality

## Code Quality
- Read existing code before modifying it
- Follow the patterns already established in the codebase
- Don't add unnecessary comments, docs, or type annotations to code you didn't change
- Don't refactor unrelated code while fixing a bug or adding a feature

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ryunzz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ryunzz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
