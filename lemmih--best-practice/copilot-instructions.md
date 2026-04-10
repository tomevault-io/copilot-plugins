## best-practice

> Universal guidelines for AI agents and developers. This file is used verbatim across multiple projects.

# Agent Guidelines

Universal guidelines for AI agents and developers. This file is used verbatim across multiple projects.

## Branches

- Work in feature branches, never directly on `main` or `master`
- Use descriptive names (e.g., `feature/add-login`, `fix/memory-leak`)

## Pull Requests

- Use `gh pr create --title "Your title" --body "Description"` to open PRs
- PR titles should follow the same Conventional Commits format as commit messages (see below)
- Link related issues when applicable

## Commits

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>[optional scope]: <description>
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`

**Examples:**
```
feat: add user authentication
fix(auth): resolve null pointer in login
docs: update installation guide
```

## Design Guidelines

### Pinned Dependencies

Pinning dependencies is vital for reproducible builds and reducing bit rot.

**Examples:**
- Using Nix for reproducible environments
- Pinning the exact channel in `rust-toolchain.toml`
- Pinning GitHub Actions runners (e.g., `ubuntu-24.04` instead of `ubuntu-latest`)
- Pinning CVE databases (used by tools like `cargo-audit`)
- Committing lock files (e.g., `package-lock.json`, `Cargo.lock`)

### Automatic Linting

Enforced linting vastly improves the quality, readability, and conformity of code. Linters should always be used and strictly enforced in CI. If a language has tooling for static analysis, it should be used.

**Examples:**
- Use `shellcheck` for shell scripts
- Use `statix` for Nix files
- Use `cargo clippy` for Rust files
- Use `hlint` for Haskell files
- Use `markdownlint` for Markdown files
- Use `yamllint` for YAML files
- Use `actionlint` for GitHub Actions workflow files
- Use `commitlint` for commit messages

### Code Formatting

Consistent code formatting improves readability and reduces friction during code reviews. Formatters should be enforced in CI alongside linters.

**Examples:**
- Use `alejandra` for Nix files
- Use `rustfmt` for Rust files
- Use `leptosfmt` for Leptos view! macros
- Use `prettier` for JavaScript/TypeScript/JSON/Markdown files
- Use `shfmt` for shell scripts

### Secure by Design

- Never build SQL with naive string interpolation—always use parameterized queries.
- Run CVE scanners (e.g., `cargo audit`) with pinned CVE databases that auto-update at least monthly.
- Validate inputs on the server; client-side checks are only for UX and must not be relied on for security.
- Keep passwords, tokens, and secrets out of git entirely (including private repos); manage them with dedicated secret-management systems.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lemmih)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lemmih)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
