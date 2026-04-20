## wealthscope

> Git workflow and commit conventions for WealthScope. Apply when creating commits, branches, or pull requests.


# Git Workflow Rules - WealthScope

## Branching Strategy

```
main        → Production (always stable)
develop     → Integration branch
feature/*   → New features (from develop)
hotfix/*    → Urgent fixes (from main)
```

## Branch Naming

```bash
# Features: feature/<short-description>
feature/auth-supabase
feature/asset-crud
feature/dashboard-ui
feature/ocr-integration
feature/what-if-engine

# Hotfixes: hotfix/<short-description>
hotfix/price-calculation
hotfix/token-refresh
```

**Rules:**
- Lowercase only
- Use hyphens, not underscores
- Keep it short but descriptive
- No spaces or special characters

## Commit Messages

### Format

```
<type>: <description>
```

### Types

| Type | When to use |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, no logic change |
| `refactor` | Code change without feature/fix |
| `test` | Adding or updating tests |
| `chore` | Maintenance, CI, dependencies |

### Good Examples

```bash
feat: add asset creation endpoint
feat: implement portfolio dashboard
fix: resolve price calculation error
fix: handle token expiration
docs: update API documentation
chore: configure GitHub Actions
test: add unit tests for asset service
refactor: extract pricing to service
```

### Bad Examples

```bash
# Too vague
fix: bug fix
feat: update

# Wrong format
Feature: Add login
FIX - resolve issue
added new stuff

# Too long
feat: add the new asset creation endpoint that handles all types of assets including stocks and real estate
```

## Workflow Commands

### Starting New Feature

```bash
git checkout develop
git pull origin develop
git checkout -b feature/my-feature
```

### Committing Changes

```bash
git add .
git commit -m "feat: description of change"
```

### Keeping Branch Updated

```bash
git fetch origin
git rebase origin/develop
```

### Creating PR

```bash
git push -u origin feature/my-feature
# Create PR on GitHub: feature/* → develop
```

### After Merge

```bash
git checkout develop
git pull origin develop
git branch -d feature/my-feature
```

## Hotfix Workflow

```bash
# Create from main
git checkout main
git pull origin main
git checkout -b hotfix/description

# Fix and commit
git commit -m "fix: description"

# Push and create PR to main
git push -u origin hotfix/description

# After merge, sync develop
git checkout develop
git merge main
git push origin develop
```

## Pre-PR Checklist

Before creating a Pull Request:

- [ ] Code compiles without errors
- [ ] Tests pass locally (`go test ./...` or `flutter test`)
- [ ] Linter passes (`golangci-lint run` or `flutter analyze`)
- [ ] Commits follow convention
- [ ] Branch is rebased on latest base
- [ ] PR has clear description

## Merge Strategy

- **feature → develop**: Squash and merge (clean history)
- **develop → main**: Merge commit (preserve history)
- **hotfix → main**: Merge commit

## What NOT to Do

```bash
# NEVER push directly to main or develop
git push origin main  # NO!

# NEVER force push to shared branches
git push --force origin develop  # NO!

# NEVER commit secrets or .env files
git add .env  # NO!

# NEVER commit large binary files
git add video.mp4  # NO!
```

## Tags for Releases

```bash
# After merging to main
git tag -a v0.1.0 -m "MVP release"
git push origin --tags
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Unikyri) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
