## defi-builder

> - **After every major feature is added**: Commit when a complete, working feature is implemented

# DeFi Builder - Cursor Rules

## Commit Guidelines

### When to Commit
- **After every major feature is added**: Commit when a complete, working feature is implemented
- **After fixing critical bugs**: Commit immediately after fixing critical issues
- **After refactoring**: Commit after significant code refactoring
- **Before breaking changes**: Commit before making breaking changes

### Commit Message Format

Use **Conventional Commits** format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

#### Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, missing semicolons, etc.)
- `refactor`: Code refactoring
- `perf`: Performance improvements
- `test`: Adding or updating tests
- `chore`: Maintenance tasks, dependency updates
- `ci`: CI/CD changes
- `build`: Build system changes

#### Scope (optional):
- `workspace`: Workspace component
- `blocks`: Block-related features
- `optimization`: Optimization engine
- `backtest`: Backtesting features
- `ui`: UI components
- `validation`: Strategy validation
- `storage`: Storage/persistence
- `api`: API integrations

#### Examples:

```
feat(workspace): add drag-and-drop block reordering

Implement drag-and-drop functionality for reordering blocks in the strategy spine.
Includes visual drop zone indicators and undo/redo support.

Closes #123
```

```
fix(backtest): fix equity curve data generation

The backtest engine was returning empty equity curves. Now properly generates
historical data points based on strategy execution.

Fixes #456
```

```
refactor(optimization): extract parameter validation logic

Move parameter validation to separate utility function for better testability
and reusability across optimization algorithms.
```

### Commit Best Practices

1. **One logical change per commit**: Each commit should represent a single, complete change
2. **Write clear, descriptive messages**: The subject should be concise but descriptive
3. **Use imperative mood**: "Add feature" not "Added feature" or "Adds feature"
4. **Reference issues**: Use "Closes #123" or "Fixes #456" in footer
5. **Keep commits focused**: Don't mix unrelated changes
6. **Test before committing**: Ensure code passes tests and linting

### Pre-Commit Checklist

Before committing, ensure:
- [ ] Code passes type checking (`npm run type-check`)
- [ ] Code passes linting (`npm run lint`)
- [ ] All tests pass (`npm test`)
- [ ] No console.logs or debug code left
- [ ] No commented-out code
- [ ] Commit message follows conventional commits format

### Branch Naming

Use descriptive branch names:
- `feat/block-reordering`
- `fix/backtest-equity-curve`
- `refactor/optimization-engine`
- `docs/api-documentation`

### Pull Request Guidelines

When creating a PR:
1. **Clear title**: Use conventional commit format
2. **Description**: Explain what, why, and how
3. **Link issues**: Reference related issues
4. **Screenshots**: Include for UI changes
5. **Testing**: Describe how to test the changes
6. **Checklist**: Use PR template checklist

### Major Feature Definition

A "major feature" is considered complete when:
- ✅ Feature is fully implemented
- ✅ All edge cases are handled
- ✅ Error handling is in place
- ✅ Basic tests are written (if applicable)
- ✅ Documentation is updated (if needed)
- ✅ Code is reviewed (if working in team)

Examples of major features:
- Complete block reordering system
- Full backtest engine implementation
- Complete optimization panel UI
- Strategy library with save/load
- Wallet integration
- Real blockchain data fetching

### Auto-Commit Reminder

When you complete a major feature, Cursor should remind you to commit with:
- A clear, conventional commit message
- Proper type and scope
- Descriptive body explaining the change

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RahilBhavan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
