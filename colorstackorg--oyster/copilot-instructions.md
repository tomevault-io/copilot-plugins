## oyster

> How to structure new pull requests


# Pull Request Guidelines

## PR Title Requirements

### Naming Convention
- **ALWAYS** use conventional commit prefixes:
  - `feat`: A non-breaking change which adds functionality
  - `fix`: A non-breaking change which fixes an issue
  - `refactor`: A change that neither fixes a bug nor adds a feature
  - `docs`: A change only to in-code or markdown documentation
  - `test`: A change that adds missing tests
  - `chore`: A change that is likely none of the above
- **ALWAYS** use lowercase letters
- **ALWAYS** start with a verb (e.g., "add ...", "implement ...", "update ...")
- **ALWAYS** include an emoji at the end (e.g., 🔥, ✨, 🐛, 📝, 🧪, 🔧)

### PR Title Examples
- `feat: add user authentication system 🔐`
- `fix: resolve database connection timeout 🐛`
- `refactor: improve error handling in API routes 🔧`
- `docs: update contributing guidelines 📝`
- `test: add unit tests for user service 🧪`
- `chore: update dependencies to latest versions ⬆️`

## PR Description Requirements

### Using the PR Template
- **ALWAYS** use the provided PR template (`.github/pull_request_template.md`)
- **ALWAYS** fill out all sections completely:
  - **Description**: Clear explanation of what the PR does with bullet points
  - **Type of Change**: Check the appropriate box
  - **Checklist**: Complete all self-review items

### Description Content
- **ALWAYS** include `Closes #<ISSUE_NUMBER>` if the PR addresses an issue
- **ALWAYS** provide a clear, concise description of changes
- **ALWAYS** use bullet points for better readability
- **ALWAYS** include screenshots for UI changes
- **ALWAYS** mention any breaking changes or important notes

## PR Process Requirements
- **ALWAYS** target the `main` branch
- **ALWAYS** ensure all CI checks pass before requesting review
- **ALWAYS** keep branches up to date with main before merging
- **ALWAYS** resolve all conversations before merging
- **ALWAYS** wait for maintainer review and approval

---
> Source: [colorstackorg/oyster](https://github.com/colorstackorg/oyster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
