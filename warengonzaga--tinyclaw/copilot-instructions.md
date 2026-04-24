## tinyclaw

> When generating commit messages for this repository, follow the **Clean Commit** workflow.

# Clean Commit Workflow

When generating commit messages for this repository, follow the **Clean Commit** workflow.

Reference: https://github.com/wgtechlabs/clean-commit

## Format

```text
<emoji> <type>: <description>
<emoji> <type> (<scope>): <description>
<emoji> <type>!: <description>
<emoji> <type>! (<scope>): <description>
```

## The 9 Types

| Emoji | Type | What it covers |
|:-----:|------|----------------|
| 📦 | `new` | Adding new features, files, or capabilities |
| 🔧 | `update` | Changing existing code, refactoring, improvements |
| 🗑️ | `remove` | Removing code, files, features, or dependencies |
| 🔒 | `security` | Security fixes, patches, vulnerability resolutions |
| ⚙️ | `setup` | Project configs, CI/CD, tooling, build systems |
| ☕ | `chore` | Maintenance tasks, dependency updates, housekeeping |
| 🧪 | `test` | Adding, updating, or fixing tests |
| 📖 | `docs` | Documentation changes and updates |
| 🚀 | `release` | Version releases and release preparation |

## Rules

- **Every single commit** must follow this format — including the very first commit (e.g. initial plan, scaffold, or setup)
- Never use plain messages like `"Initial plan"` or `"WIP"` — always use the Clean Commit format
- Use lowercase for type
- Use `!` immediately after type (no space) to signal a breaking change — only for `new`, `update`, `remove`, `security`
- Use present tense ("add" not "added")
- No period at the end
- Keep description under 72 characters

## Examples

- `📦 new: user authentication system`
- `🔧 update (api): improve error handling`
- `🗑️ remove (deps): unused lodash dependency`
- `🔒 security: patch XSS vulnerability`
- `⚙️ setup: add eslint configuration`
- `☕ chore: update npm dependencies`
- `🧪 test: add unit tests for auth service`
- `📖 docs: update installation instructions`
- `🚀 release: version 1.0.0`
- `📦 new!: completely redesign authentication system`
- `🔧 update! (api): change response format for all endpoints`

---
> Source: [warengonzaga/tinyclaw](https://github.com/warengonzaga/tinyclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
