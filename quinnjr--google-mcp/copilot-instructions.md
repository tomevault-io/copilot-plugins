## git-flow

> Git-flow branching model and commit conventions for this repository


# Git-Flow Workflow

This repository follows the git-flow branching model.

## Branch Types

| Branch | Purpose | Naming Convention |
|--------|---------|-------------------|
| `main` | Production-ready code | Protected, only merges from release/hotfix |
| `develop` | Integration branch for features | Protected |
| `feature/*` | New features | `feature/short-description` |
| `bugfix/*` | Bug fixes for develop | `bugfix/short-description` |
| `release/*` | Release preparation | `release/vX.Y.Z` |
| `hotfix/*` | Production emergency fixes | `hotfix/short-description` |

## Commit Message Format

Use conventional commits format:

```
<type>(<scope>): <subject>

[optional body]

[optional footer]
```

### Types

- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `style`: Code style (formatting, semicolons, etc.)
- `refactor`: Code refactoring (no feature/fix)
- `perf`: Performance improvement
- `test`: Adding/updating tests
- `chore`: Build process, dependencies, tooling
- `ci`: CI/CD changes

### Scopes (for this project)

- `auth`: OAuth authentication
- `drive`: Google Drive service
- `docs`: Google Docs service
- `sheets`: Google Sheets service
- `calendar`: Google Calendar service
- `gmail`: Gmail service
- `contacts`: People/Contacts service
- `youtube`: YouTube service
- `slides`: Google Slides service
- `tasks`: Google Tasks service
- `server`: MCP server core
- `types`: Type definitions

### Examples

```
feat(calendar): add recurring event support

fix(auth): handle token refresh failure gracefully

chore(deps): update googleapis to v145

docs: update README with new setup instructions
```

## Workflow

1. **Starting a feature:**
   ```bash
   git checkout develop
   git pull origin develop
   git checkout -b feature/my-feature
   ```

2. **Completing a feature:**
   ```bash
   git checkout develop
   git merge --no-ff feature/my-feature
   git push origin develop
   git branch -d feature/my-feature
   ```

3. **Creating a release:**
   ```bash
   git checkout develop
   git checkout -b release/v1.2.0
   # bump version, final fixes
   git checkout main
   git merge --no-ff release/v1.2.0
   git tag -a v1.2.0
   git checkout develop
   git merge --no-ff release/v1.2.0
   ```

4. **Hotfix:**
   ```bash
   git checkout main
   git checkout -b hotfix/critical-fix
   # fix the issue
   git checkout main
   git merge --no-ff hotfix/critical-fix
   git tag -a v1.2.1
   git checkout develop
   git merge --no-ff hotfix/critical-fix
   ```

## Rules

- Never commit directly to `main` or `develop`
- Always use `--no-ff` for merges to preserve branch history
- Delete feature branches after merging
- Tag all releases on `main`
- Write meaningful commit messages following conventional commits

---
> Source: [quinnjr/google-mcp](https://github.com/quinnjr/google-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
