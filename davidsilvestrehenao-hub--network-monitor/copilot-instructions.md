## network-monitor

> CI/CD Workflow & Pull Request Process


# CI/CD Workflow & Pull Request Process

**CRITICAL:** This project requires all changes to go through Pull Requests. Direct commits to main are blocked.

For complete CI/CD workflow guidelines, see: `.cursor/ci-workflow.md`

## Quick Summary for AI Agents

When helping with code changes:

1. **Always work on feature branches**
   - Use: `./scripts/new-feature.sh <name>`
   - Never commit directly to main
   - Follow branch naming: `<type>/<description>`

2. **Before suggesting "commit and push"**
   - Remind user to run: `./scripts/ci-check.sh`
   - This runs local CI checks (format, lint, type-check, build)
   - Help fix any issues found

3. **Creating Pull Requests**
   - Use: `gh pr create --fill`
   - PR titles must follow semantic commit format: `<type>: <Description>`
   - Valid types: feat, fix, docs, style, refactor, perf, test, build, ci, chore

4. **Automated Reviews**
   - GitHub Actions CI runs automatically
   - Gemini Code Assist reviews all PRs
   - User can interact with: `/gemini review`, `/gemini summary`, `@gemini-code-assist`

5. **Required to Pass**
   - ✅ Code formatting (Prettier)
   - ✅ Linting (ESLint)
   - ✅ Type checking (TypeScript)
   - ✅ Build verification
   - ✅ PR title validation
   - ✅ Branch name validation
   - ✅ No merge conflicts
   - ✅ No secrets leaked

6. **Never Suggest**
   - ❌ Committing directly to main
   - ❌ Using `--no-verify` or `--force`
   - ❌ Disabling CI checks
   - ❌ Bypassing branch protection

## Architecture Enforcement

When reviewing or writing code:

- **Router → Service → Repository pattern** must be followed
- **Event-driven communication** between services (no direct service calls)
- **Dependency injection** for all services
- **TypeScript type safety** - no `any` types without justification
- **12-Factor App compliance** - config via environment variables
- **Code quality** - zero ESLint warnings/errors

## Complete Workflow Example

```bash
# 1. Create feature branch
./scripts/new-feature.sh add-user-auth

# 2. Make changes
# ... edit files ...

# 3. Commit
git add .
git commit -m "feat: Add user authentication"

# 4. Run local CI checks
./scripts/ci-check.sh

# 5. Push
git push origin feature/add-user-auth

# 6. Create PR
gh pr create --fill

# 7. Wait for automated reviews
# - GitHub Actions CI
# - Gemini Code Assist

# 8. Merge when approved
gh pr merge --squash --delete-branch
```

## Helping with CI Failures

### Formatting Issues
```bash
bun run format
git add .
git commit -m "style: Fix formatting"
```

### Linting Issues
```bash
bun run lint:fix
git add .
git commit -m "style: Fix linting"
```

### Type Errors
```bash
bun run type-check
# Help user fix type errors
git add .
git commit -m "fix: Resolve type errors"
```

## References

- Full CI/CD workflow: `.cursor/ci-workflow.md`
- GitHub workflows: `.github/workflows/`
- Gemini config: `.gemini/config.yml`
- Helper scripts: `scripts/`

---

**Key Principle:** All AI agents must guide users through the proper PR workflow and never suggest bypassing CI/CD checks or branch protection rules.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidsilvestrehenao-hub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
