## 05-dependencies

> Supply chain security rules for dependency management.


# Dependency & Supply Chain Security

## Adding Dependencies

- NEVER add new dependencies without user approval.
- When proposing, include: name, version, purpose, popularity, license, and known vulnerabilities.
- Prefer well-established, widely-used packages.
- Pin exact versions in lock files.
- Verify package names carefully to avoid typosquatting.

## Lock Files

- NEVER delete or regenerate lock files without explicit approval.
- Lock file changes must be reviewed carefully.
- Resolve conflicts in lock files carefully — never simply regenerate.

## Vulnerability Response

- If you discover a known vulnerability, warn the user immediately.
- Propose the minimal update needed to resolve it.
- Never ignore or suppress vulnerability warnings.

## Package Verification

- Check that package sources match expected repositories.
- Be suspicious of packages with very few downloads or very new publish dates.
- Never install packages from URLs, git repos, or local paths without review.

---
> Source: [the-missing-pink/ai-repository-security-baseline](https://github.com/the-missing-pink/ai-repository-security-baseline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
