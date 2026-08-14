## dev-tools

> FastForward\DevTools is a Composer plugin that aggregates multiple PHP development tools into a single unified command. It provides automated execution of tests, static analysis, code styling, refactoring, and documentation generation.

# AGENTS.md

## Project Overview

FastForward\DevTools is a Composer plugin that aggregates multiple PHP development tools into a single unified command. It provides automated execution of tests, static analysis, code styling, refactoring, and documentation generation.

**Key Technologies:**

- PHP 8.3+
- Composer plugin
- PHPUnit 12.5 (testing)
- Rector 2.x (automated refactoring)
- Easy Coding Standard (code style)
- phpDocumentor (API docs)
- GrumPHP (Git hooks)
- PHPSpec/Prophecy (test doubles)

## Setup Commands

```bash
# Install as development dependency
composer require --dev fast-forward/dev-tools:dev-main

# Install dependencies
composer install
```

## Development Workflow

```bash
# Run all standard checks (refactoring, code styling, docs, tests, reports)
composer dev-tools

# Automatically fix code standards issues
composer dev-tools:fix

# Individual commands
composer dev-tools tests         # Run PHPUnit tests
composer dev-tools code-style    # Check and fix code style (ECS + Composer Normalize)
composer dev-tools refactor      # Refactor code using Rector
composer dev-tools phpdoc        # Check and fix PHPDoc comments
composer dev-tools docs          # Generate HTML API documentation
composer dev-tools wiki          # Generate Markdown documentation for wiki
composer dev-tools reports       # Generate docs frontpage and reports
composer dev-tools agents        # Sync packaged project agents into .agents/agents
composer dev-tools:sync          # Sync managed repository assets and packaged agent surfaces
```

**Notable Specialized Commands:**

- `composer skills`: Synchronize packaged skills into consumer `.agents/skills`
- `composer funding`: Sync funding metadata between `composer.json` and `.github/FUNDING.yml`
- `composer codeowners`: Generate `.github/CODEOWNERS` from repository metadata
- `composer gitattributes`: Manage export-ignore rules in `.gitattributes`
- `composer gitignore`: Synchronize managed `.gitignore` content
- `composer license`: Generate or refresh repository `LICENSE` files
- `composer dependencies`: Run dependency analysis workflows
- `composer metrics`: Generate PhpMetrics reports and related artifacts
- `composer update-composer-json`: Normalize managed `composer.json` settings
- `composer changelog:entry|check|next-version|promote|show`: Manage changelog-driven release workflows
- `composer dev-tools:sync --dry-run|--check|--interactive`: Preview managed-file drift while intentionally skipping wiki, skills, and agents

## Testing Instructions

```bash
# Run all tests with coverage
composer dev-tools tests

# Run tests matching a pattern
composer dev-tools tests -- --filter=CodeStyle

# Run with coverage report
composer dev-tools tests -- --coverage=.dev-tools/coverage
```

**Test Configuration:**

- PHPUnit XML config: `phpunit.xml`
- Test namespace: `FastForward\DevTools\Tests\`
- Source namespace: `FastForward\DevTools\`
- Coverage required (strict metadata enforcement)
- Coverage threshold: configured in `phpunit.xml`

**Test Patterns:**

- Tests use PHPUnit 12.x with Prophecy for mocking
- Custom PHPUnit extension: `FastForward\DevTools\PhpUnit\Runner\Extension\DevToolsExtension`

**Creating/Updating Tests:**

- Use skill `phpunit-tests` in `.agents/skills/phpunit-tests/` for creating or updating PHPUnit tests with Prophecy
- Run skill when: creating new test classes, adding test methods, or fixing existing tests

## Code Style

**PHP Coding Standard:** PSR-12 with Symfony style

**ECS Configuration:** `ecs.php`

- Uses Symfony, PSR-12, and Symplify rule sets
- PHPDoc alignment: left-aligned
- PHPUnit test case static methods: use `self::`
- Skips: vendor, resources, public, tmp directories

**File Organization:**

- Source code: `src/` (PSR-4: `FastForward\DevTools\`)
- Tests: `tests/` (PSR-4: `FastForward\DevTools\Tests\`)
- Commands: `src/Console/Command/`
- PHPUnit events: `src/PhpUnit/Event/`
- Rector rules: `src/Rector/`
- Composer plugin: `src/Composer/`

**Architecture Direction:**

- Avoid introducing new dependencies on `composer/composer` outside the
  existing Composer plugin integration and legacy surfaces already awaiting
  decoupling.
- Prefer DevTools-owned interfaces for generic runtime concerns such as
  environment variables, process execution, filesystem access, and console
  presentation instead of reaching for Composer utility classes.
- When an existing Composer utility is convenient, first check whether a small
  local abstraction would support the ongoing Composer decoupling work with
  minimal code.
- During the Composer-to-Symfony command migration, preserve the global
  execution affordances already relied on by nested tools: `--ansi`/`--no-ansi`,
  cache and `--cache-dir` handling, and working-directory behavior.
- Keep color behavior explicit in command wrappers for tools with known flags
  instead of probing binaries dynamically; for example, Symfony/Composer-style
  tools can receive `--ansi`, while PHPUnit uses `--colors=always`.
- Keep child-process environment policy centralized in `ProcessQueue`
  configurators, including disabling Xdebug for non-coverage subprocesses while
  preserving coverage drivers when PCOV is unavailable.
- Prefer the project-approved `Safe\*` function imports whenever replacing or
  adding native PHP calls that have Safe equivalents, so Rector does not have
  to rewrite them during commit checks.

**Naming Conventions:**

- Classes: PascalCase
- Methods: camelCase
- Properties: camelCase
- Constants: UPPER_CASE

**PHPdoc Requirements:**

- All classes require docblocks with `@copyright` and `@license`
- Use RFC 2119 keywords (MUST, SHOULD, MAY, etc.)
- PHPDoc checked via dev-tools phpdoc command
- Use skill `phpdoc-code-style` in `.agents/skills/phpdoc-code-style/` for PHPDoc cleanup and repository-specific PHP formatting

## Build and Deployment

This repository ships a Composer plugin plus the local `bin/dev-tools` binary,
so there is no separate frontend or asset build step. The package is published
to Packagist, while consumer repositories adopt workflows, packaged skills, and
other managed assets through `composer dev-tools:sync`.

Release and publishing behavior is driven primarily through
`.github/workflows/tests.yml`, `changelog.yml`, `reports.yml`, `wiki.yml`,
`wiki-preview.yml`, `wiki-maintenance.yml`, `auto-assign.yml`, and
`label-sync.yml`, with reusable local workflow building blocks grouped under
`.github/actions/` and packaged consumer workflow wrappers living under
`resources/github-actions/`. Packaged skills live under `.agents/skills/`
alongside mirrored project-agent prompts under `.agents/agents/`.

**Package Details:**

- Type: `composer-plugin`
- Class: `FastForward\DevTools\Composer\Plugin`
- Minimum stability: `stable`

## Pull Request Guidelines

**Required Checks:**

```bash
composer dev-tools
```

**Before Submitting:**

- Run full dev-tools suite
- Ensure all tests pass
- Code style must be clean (ECS)
- PHPDoc must be valid
- Coverage must meet threshold
- Rector should handle refactoring automatically
- Follow `.github/pull_request_template.md` and include issue linkage,
  verification notes, documentation/generated-output review, and changelog
  status

## Canonical References

- `README.md`: high-level command surface, architecture overview, and consumer-facing context
- `docs/commands/`: command-specific behavior and option details
- `docs/usage/` and `docs/internals/`: workflow, reporting, release, and implementation notes
- `.github/workflows/`: CI and release automation truth, especially `tests.yml`, `reports.yml`, `review.yml`, `wiki.yml`, `wiki-preview.yml`, `wiki-maintenance.yml`, `changelog.yml`, `auto-assign.yml`, and `label-sync.yml`
- `.github/actions/`: shared workflow building blocks for `php`, `project-board`, `github-pages`, `review`, `summary`, `wiki`, `changelog`, and `label-sync`
- `resources/github-actions/`: consumer-facing workflow wrappers synchronized by `dev-tools:sync`
- `.github/pull_request_template.md`: expected PR structure and reviewer checklist
- `src/Sync/`: shared packaged-directory synchronization primitives used by `skills` and `agents`
- `.agents/skills/`: packaged procedural skills shipped to consumer repositories
- `.agents/agents/`: repository-specific role prompts mirrored through `.github/agents`
- `.github/wiki`: generated or synchronized wiki content published by repo workflows

## Additional Notes

- **GrumPHP**: Automatically runs on pre-commit (configured in `grumphp.yml`)
- **Rector**: Custom rules in `src/Rector/` for automated refactoring
- **Documentation**: Sphinx-based docs in `docs/` directory
- **Wiki**: Pull-request preview publication starts in `.github/workflows/wiki.yml`, while merged publication and preview cleanup run through `.github/workflows/wiki-maintenance.yml`
- **GitHub Actions**: Reusable workflows live in `.github/workflows/`, local workflow actions live in `.github/actions/`, and consumer wrappers are synchronized from `resources/github-actions/`
- **Dependency Health CI**: `.github/workflows/tests.yml` always runs the dependency-health job, and its default `max-outdated` input is `-1` so outdated packages are reported without failing CI on count alone
- **Sync Preview Modes**: `dev-tools:sync --dry-run`, `--check`, and `--interactive` intentionally skip wiki, skills, and agents because those flows do not yet expose non-destructive verification
- **Project Agents**: Packaged role prompts synchronized via `composer agents` and `dev-tools:sync`

## Skills Usage

- **Creating/Updating Tests**: Use skill `phpunit-tests` in `.agents/skills/phpunit-tests/` for PHPUnit tests with Prophecy
- **Creating/Refreshing AGENTS.md**: Use skill `create-agentsmd` in `.agents/skills/create-agentsmd/` to generate or update repository-root AGENTS instructions for coding agents
- **Generating Documentation**: Use skill `sphinx-docs` in `.agents/skills/sphinx-docs/` for Sphinx documentation in `docs/`
- **Updating README**: Use skill `package-readme` in `.agents/skills/package-readme/` for generating README.md files
- **Updating PHPDoc / PHP Style**: Use skill `phpdoc-code-style` in `.agents/skills/phpdoc-code-style/` for PHPDoc cleanup and repository-specific PHP formatting
- **Drafting / Publishing GitHub Issues**: Use skill `github-issues` in `.agents/skills/github-issues/` to transform a short feature description into a complete, production-ready GitHub issue and create or update it on GitHub when needed
- **Implementing Issues & PRs**: Use skill `github-pull-request` in `.agents/skills/github-pull-request/` to iterate through open GitHub issues and implement them one by one with branching, testing, documentation, and pull requests
- **Reviewing Pull Requests**: Use skill `pull-request-review` in `.agents/skills/pull-request-review/` for findings-first Fast Forward pull-request review focused on regressions, missing tests, missing docs, workflow risk, and generated-output drift

## Project Agents

Packaged project-agent prompts live in `.agents/agents/` for both this
repository and consumer repositories that synchronize DevTools assets. These
role prompts define behavior and ownership boundaries, while `.agents/skills/`
remains the procedural source of truth.

- Use `issue-editor` for issue drafting, refinement, comments, updates, and closure workflows.
- Use `issue-implementer` for issue-to-branch-to-PR execution.
- Delegate to `agents-maintainer` when repository `AGENTS.md` guidance needs to be created, refreshed, or realigned with current workflows.
- Delegate to `test-guardian` whenever behavior changes, regressions, or missing coverage are involved.
- Delegate to `php-style-curator` for PHPDoc cleanup, file-header normalization, and repository style conformance.
- Delegate to `readme-maintainer` when public commands, installation, usage, links, or badges change.
- Delegate to `docs-writer` when `docs/` must be created or updated.
- Delegate to `consumer-sync-auditor` when packaged skills, packaged agents, sync assets, wiki, workflow wrappers, local GitHub actions, or consumer bootstrap behavior change.
- Delegate to `quality-pipeline-auditor` when a task changes command orchestration, verification flow, or quality gates.
- Delegate to `review-guardian` when a pull request needs a rigorous findings-first review before or during human review.
- Delegate to `changelog-maintainer` when a task needs changelog authoring, changelog validation for PRs, release promotion, or release-note export.

---
> Source: [php-fast-forward/dev-tools](https://github.com/php-fast-forward/dev-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
