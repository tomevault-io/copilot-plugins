## sentry-cli-flake

> This project uses [Conventional Commits](https://www.conventionalcommits.org/).

# Agents Guide

## Commit Convention

This project uses [Conventional Commits](https://www.conventionalcommits.org/).

Format: `<type>: <description>`

Common types:
- `feat` – new feature
- `fix` – bug fix
- `chore` – maintenance, dependency updates, tooling
- `docs` – documentation only
- `refactor` – code restructuring without behavior change
- `ci` – CI/CD pipeline changes
- `test` – adding or updating tests
- `style` – formatting, whitespace

Examples from this repo:
```
feat: add support for all major Nix platforms
fix: correct override params for multi-platform
chore: update Sentry CLI version
docs: add nix run from GitHub instructions to README
ci: add auto-updating workflow
```

Keep the description lowercase and concise. No period at the end.

When the change is non-trivial, add a body to summarize what changed and why. Separate it from the subject with a blank line:

```
feat: add support for all major Nix platforms

Added platform-specific packages for x86_64-linux, aarch64-linux,
x86_64-darwin, and aarch64-darwin. Each platform pulls the correct
Sentry CLI binary from upstream.
```

---
> Source: [KennethHoff/sentry-cli-flake](https://github.com/KennethHoff/sentry-cli-flake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
