## commit-messages

> Use the Conventional Commits (CC) format for all commit messages:

# Commit Message Format

## Conventional Commits Format
Use the Conventional Commits (CC) format for all commit messages:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Common Types:
- `feat`: A new feature
- `fix`: A bug fix
- `docs`: Documentation only changes
- `style`: Changes that do not affect the meaning of the code
- `refactor`: A code change that neither fixes a bug nor adds a feature
- `perf`: A code change that improves performance
- `test`: Adding missing tests or correcting existing tests
- `chore`: Changes to the build process or auxiliary tools

### Examples:
- `feat(auth): add user login functionality`
- `fix: resolve navigation bug on mobile`
- `docs: update README installation instructions`
- `refactor(components): simplify button component logic`

## AI Agent Commits
For commits made by AI agents with no user input, prefix the commit message with `[agent]`:

```
[agent] <type>[optional scope]: <description>
```

### Examples:
- `[agent] feat(ui): generate responsive navigation component`
- `[agent] fix(api): resolve type errors in user endpoints`
- `[agent] refactor: optimize database queries for performance`

## Guidelines:
- Keep the description concise but descriptive
- Use lowercase for the description
- Don't end the description with a period
- Use the body to explain the "what" and "why" vs. "how"
- Reference issues and pull requests in the footer when applicable

---
> Source: [intern3-chat/intern3-chat](https://github.com/intern3-chat/intern3-chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
