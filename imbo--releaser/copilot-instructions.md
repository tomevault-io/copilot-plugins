## releaser

> Guidance for coding agents working in this repository.

# AGENTS.md

Guidance for coding agents working in this repository.

## Purpose

`imbo/releaser` is a PHP 8.3+ CLI application that helps create GitHub releases by:

- Inspecting branches, tags, and merged pull requests using the GitHub API
- Determining the next semantic version from Conventional Commit pull request titles
- Creating tags/releases via GitHub APIs

## Tech stack

- Language: PHP 8.3+
- CLI framework: Symfony Console
- HTTP: Guzzle
- SemVer: `phlak/semver`
- Conventional Commits parser: `ramsey/conventional-commits`
- Tests: PHPUnit
- Static analysis: PHPStan level 10
- Formatting/linting: PHP-CS-Fixer via `imbo/imbo-coding-standard`

## Repository map

- `imbo-releaser`: CLI entrypoint script
- `src/Command/Release.php`: Main release workflow command
- `src/Console/Application.php`: Symfony Console app wiring
- `src/Config.php`, `src/ConfigInterface.php`, `src/Config/Resolver.php`: configuration defaults and config file loading
- `src/GitHub/*`: API client and DTOs
- `tests/*`: PHPUnit tests

## Local workflow commands

Run from repository root:

- `composer run test` — run PHPUnit
- `composer run sa` — run PHPStan
- `composer run cs` — check coding style
- `composer run cs:fix` — auto-fix coding style
- `composer run ci` — check coding style + run static analysis and tests

Run `composer run ci` after making changes.

## Coding conventions

- Keep `declare(strict_types=1);` in PHP files.
- Match existing namespace/layout conventions (`ImboReleaser\...`, PSR-4 under `src/`).
- Prefer typed signatures and explicit return types.
- Keep docblocks for non-obvious behavior and useful type information (especially array shapes/generics for PHPStan).
- Follow existing error handling style: throw `RuntimeException` / `InvalidArgumentException` with actionable messages.
- Keep changes surgical; avoid broad refactors unless explicitly requested.

## Testing guidance

- Add/update PHPUnit tests when modifying behavior in `src/`.
- Prefer focused tests next to related existing test suites (`tests/Config`, `tests/GitHub`, etc.).
- Use `#[CoversClass(...)]` like existing tests.
- Do not introduce network-dependent tests; use Guzzle mock handlers/patterns already used in `tests/GitHub/ClientTest.php`.

## Domain rules to preserve

- Repository format is `owner/repo`.
- Branch conventions are centered around `main` and maintenance branches (`X.x` / `vX.x`) per README assumptions.
- Tag conventions are semantic versions (`X.Y.Z`, optional `v` prefix).
- Configuration is loaded from `.imbo-releaser.php` or `.imbo-releaser.dist.php` (or explicit `--config`).

## Guardrails

- Do not edit `vendor/` or generated coverage output under `build/`.
- Avoid committing local environment files (`.env`, `.imbo-releaser.php`, local tool cache files).
- Keep CLI UX and options backward-compatible unless the task explicitly requires breaking changes.
- If you see unrelated local changes in the working tree, do not revert them.

## When implementing new features

1. Update source in `src/` with minimal surface-area changes.
2. Add/adjust tests in `tests/`.
3. Run relevant checks (`composer test`, `composer sa`, `composer cs`).
4. Update `README.md` when user-facing behavior or configuration changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imbo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/imbo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
