## git

> - Always use Conventional Commits format for all commit messages


# Git & Version Control Rules

## Commit Message Style (Conventional Commits)
- Always use Conventional Commits format for all commit messages
- Format: `<type>(<scope>): <subject>`
- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`
- Scope is optional but recommended (e.g., `feat(auth): add login form`)
- Subject should be lowercase, imperative mood, no period at the end
- Use body for detailed explanation when needed
- Use footer for breaking changes: `BREAKING CHANGE: <description>`

## Commit Message Examples
```
feat(auth): add user login functionality
fix(api): resolve timeout issue in request service
docs(readme): update installation instructions
style(ui): format button component
refactor(entities): extract common validation logic
perf(queries): optimize React Query cache settings
test(forms): add validation tests for signup form
chore(deps): update dependencies to latest versions
ci(github): add automated testing workflow
build(docker): configure production build
```

## Commit Best Practices
- Write clear, descriptive commit messages
- Keep commits focused and atomic (one logical change per commit)
- Use meaningful branch names following pattern: `type/description` (e.g., `feat/user-profile`, `fix/auth-bug`)
- Review code before committing (check for console.logs, debug code, etc.)
- Never commit sensitive data (API keys, passwords, tokens)
- Never commit generated files or build artifacts
- Use `.gitignore` properly to exclude unnecessary files

## Pre-commit Requirements
- **ALWAYS run tests before committing**: `npm test` or `npm run test`
- **ALWAYS run linter before committing**: `npm run lint` or `npm run lint:fix`
- **ALWAYS check TypeScript types**: `npm run type-check` or `tsc --noEmit`
- Fix all linting errors and failing tests before committing
- Ensure code builds successfully: `npm run build`

## Branch Management
- Use descriptive branch names: `feat/feature-name`, `fix/bug-name`, `refactor/component-name`
- Keep branches up-to-date with main branch: `git pull origin main` before creating PR
- Delete branches after merging
- Use feature branches for all new work, never commit directly to `main` or `develop`

## Pull Request Best Practices
- Create PRs for all changes, even small ones
- Write clear PR descriptions explaining what and why
- Link related issues in PR description
- Ensure all CI checks pass before requesting review
- Request review from at least one team member
- Address review comments before merging
- Keep PRs focused and reasonably sized (avoid huge PRs)

## Git Workflow
- Pull latest changes before starting work: `git pull origin main`
- Create feature branch: `git checkout -b feat/feature-name`
- Make small, frequent commits
- Push branch regularly: `git push origin feat/feature-name`
- Rebase or merge main into feature branch before finalizing PR
- Squash commits when appropriate (keep history clean)

## Code Review Checklist
- Review your own code before submitting PR
- Check for common mistakes and patterns
- Ensure consistency with codebase
- Verify FSD import rules are followed
- Check for console.logs, debug code, commented code
- Verify all tests pass
- Ensure no TypeScript errors
- Check for proper error handling
- Verify accessibility requirements
- Test the changes manually

## Git Hooks (Recommended Setup)
- Set up pre-commit hook to run linter and tests automatically
- Use tools like `husky` and `lint-staged` for automated checks
- Configure pre-push hook to run full test suite
- Example setup:
  ```json
  {
    "husky": {
      "hooks": {
        "pre-commit": "lint-staged",
        "pre-push": "npm test"
      }
    },
    "lint-staged": {
      "*.{ts,tsx}": ["eslint --fix", "prettier --write"]
    }
  }
  ```

## File Management
- Never commit `.env` files or environment-specific configs
- Use `.env.example` for documenting required environment variables
- Never commit `node_modules/`, `dist/`, `build/` directories
- Keep `.gitignore` up-to-date
- Commit lock files (`package-lock.json`, `yarn.lock`) for consistency

## Conflict Resolution
- Resolve conflicts carefully, understand both sides
- Test thoroughly after resolving conflicts
- Don't commit conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`)
- Communicate with team when conflicts are complex

## Commit Frequency
- Commit often with small, logical changes
- Don't accumulate many changes in one commit
- Each commit should represent a working state of the codebase
- Use `git add -p` for selective staging when needed

---
> Source: [AndriiMelnikk/hochu_next](https://github.com/AndriiMelnikk/hochu_next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
