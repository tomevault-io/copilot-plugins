## gitflow

> Gitflow Workflow Rules. These rules should be applied when performing git operations.

# Gitflow Workflow Rules

## Main Branches

### main (or master)
- Contains production-ready code
- Never commit directly to main
- Only accepts merges from:
  - hotfix/* branches
  - release/* branches
- Must be tagged with version number after each merge

### develop
- Main development branch
- Contains latest delivered development changes
- Source branch for feature branches
- Never commit directly to develop

## Supporting Branches

### feature/*
- Branch from: develop
- Merge back into: develop
- Naming convention: feature/[issue-id]-descriptive-name  
  Example: feature/123-user-authentication
- Must be up-to-date with develop before creating PR
- Delete after merge

### release/*
- Branch from: develop
- Merge back into: 
  - main
  - develop
- Naming convention: release/vX.Y.Z  
  Example: release/v1.2.0
- Only bug fixes, documentation, and release-oriented tasks (no new features)
- Delete after merge

### hotfix/*
- Branch from: main
- Merge back into:
  - main
  - develop
- Naming convention: hotfix/vX.Y.Z  
  Example: hotfix/v1.2.1
- Only for urgent production fixes
- Delete after merge

## Commit Messages

- Format: `type(scope): description`
- Types:
  - feat: New feature
  - fix: Bug fix
  - docs: Documentation changes
  - style: Formatting or syntax fixes
  - refactor: Code refactoring
  - test: Adding tests
  - chore: Maintenance tasks
- Keep commits atomic and descriptive
- Use conventional commits (feat/fix/chore)

## Version Control

### Semantic Versioning
- **MAJOR**: Incompatible API changes  
- **MINOR**: Backwards-compatible functionality  
- **PATCH**: Backwards-compatible bug fixes

## Pull Request Rules

1. All changes must go through Pull Requests  
2. Minimum one approval required  
3. CI checks must pass  
4. No direct commits to protected branches (main, develop)  
5. Branch must be up to date before merging  
6. Delete branch after merge  

## Branch Protection Rules

### main & develop
- Require pull request reviews  
- Require status checks to pass  
- Require branches to be up to date  
- Include administrators in restrictions  
- No force pushes  
- No deletions  

## Repository Hygiene

- Fetch often, rebase before push  
- No merge commits; use fast-forward only  
- Do not commit secrets or sensitive data  
- No force-push on shared branches  
- Delete branches only after merge unless otherwise requested  
- CI/CD must pass before merge  

## Release Process

1. Create release branch from develop  
2. Bump version numbers  
3. Fix release-specific issues  
4. Create PR to main  
5. After merge to main:  
   - Tag release  
   - Merge back to develop  
   - Delete release branch  

## Hotfix Process

1. Create hotfix branch from main  
2. Fix the issue  
3. Bump patch version  
4. Create PR to main  
5. After merge to main:  
   - Tag release  
   - Merge back to develop  
   - Delete hotfix branch  

---
> Source: [MylesThomas/betting](https://github.com/MylesThomas/betting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
