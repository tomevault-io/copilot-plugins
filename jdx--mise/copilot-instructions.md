## mise

> how to write commit and PR titles


## Conventional Commits (REQUIRED)

All commit messages and PR titles MUST follow conventional commit format:

### Format
```
<type>(<scope>): <description>
```

### Types
- `feat:` - New features
- `fix:` - Bug fixes  
- `refactor:` - Code refactoring
- `docs:` - Documentation
- `style:` - Code style/formatting
- `perf:` - Performance improvements
- `test:` - Testing changes
- `chore:` - Maintenance tasks
- `chore(deps):` - Dependency updates

### Examples
```
feat(cli): add new command for tool management
fix(config): resolve parsing issue with nested tables
refactor(backend): simplify plugin loading logic
doc(api): update configuration examples
test(e2e): add tests for tool installation
chore(deps): update Rust dependencies
```

### Common Scopes
`registry`, `aqua`, `cli`, `config`, `backend`, `tool`, `env`, `task`, `api`, `ui`, `core`, `deps`, `schema`, `doctor`, `shim`, `security`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jdx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jdx)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
