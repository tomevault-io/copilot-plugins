## auto-commit

> Read this when user asks you to auto commit as you go.

<git-commit-guidelines>

When making changes to code, commit your changes incrementally as you work. Follow these commit conventions:

<commit-format>
```
<type>[optional scope]: <subject>

[optional body]
```
</commit-format>

<types>
- feat: New feature or functionality
- fix: Bug fix
- chore: Maintenance tasks, dependency updates, configuration changes
- docs: Documentation only changes
- style: Code style/formatting changes (no functional changes)
- refactor: Code restructuring without changing functionality
- test: Adding or modifying tests
- perf: Performance improvements
- ci: Changes to CI configuration files and scripts
- build: Changes to build system or external dependencies
- revert: Reverts a previous commit
</types>

<rules>

<subject-line>
- Maximum 50 characters (hard limit: 72)
- Start with lowercase letter
- No period at the end
- Use imperative mood ("add" not "adds" or "added")
- Be concise but descriptive
</subject-line>

<scope-rules>
- Optional component/module/area affected (e.g., "auth", "api", "ui")
- Use lowercase
- Keep short and meaningful
- Examples: feat(auth): add login validation, fix(api): handle null responses
</scope-rules>

<body-rules>
- Separate from subject with blank line
- Wrap lines at 72 characters for readability
- Explain what and why, not how
- Only include when the change requires additional context
- Use bullet points for multiple changes if needed
</body-rules>

<commit-frequency>
- Commit after each logical unit of change
- Each commit should represent one coherent change
- Don't bundle unrelated changes
</commit-frequency>
</rules>

<examples>

<example type="good">
```
feat(auth): add user authentication middleware

Implements JWT-based authentication for API routes.
Includes token validation and refresh logic.
```
</example>

<example type="good">
```
fix(api): resolve null pointer in user lookup
```
</example>

<example type="good">
```
chore: update dependencies to latest versions
```
</example>

<example type="good">
```
docs(readme): add installation instructions
```
</example>

<example type="bad">
```
feat: added new feature to the application that allows users to authenticate using JWT tokens and also fixed some bugs and updated dependencies
```
Reason: Too long, bundles multiple unrelated changes, uses past tense
</example>

<example type="bad">
```
fix: Fixed bug.
```
Reason: Capitalized subject, vague description, ends with period
</example>

<example type="bad">
```
Update stuff
```
Reason: Missing type, vague, capitalized
</example>
</examples>

<key-principles>
- Be concise but descriptive
- One commit = one logical change
- Commit message should make sense without looking at the code
- Skip the body if the subject line is self-explanatory
- Use present tense, imperative mood
- Focus on what the change does, not what you did
- Make each commit atomic and reversible
</key-principles>

<workflow-tips>
- Stage related changes together: `git add <files>`
- Review changes before committing: `git diff --staged`
- Use `git commit -m "message"` for simple commits
- Use `git commit` (without -m) for commits needing a body
- Amend the last commit if needed: `git commit --amend`
</workflow-tips>

</git-commit-guidelines>

---
> Source: [nextify-limited/libra](https://github.com/nextify-limited/libra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
