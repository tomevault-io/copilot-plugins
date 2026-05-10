## tarot

> <!-- TEMPLATE:START — managed by sync-template-config, do not edit manually.

# CLAUDE.md — Mandatory rules

<!-- TEMPLATE:START — managed by sync-template-config, do not edit manually.
     Add here: shared conventions (coding standards, git rules, language rules, approach,
     token optimization, recommended plugins, translations).
     If unsure whether a rule belongs here or in PROJECT, ask the user before adding. -->

## Approach
- Act over ask; read only files you'll edit.
- Key context: CLAUDE.md, MEMORY.md, `docs/patterns.md` — update `docs/patterns.md` when adding entities/controllers/forms/commands/routes.
- CLAUDE.md and `docs/patterns.md` must be optimized for LLM use and token efficiency, without loss of information/instruction.
- Complex tasks: plan → approval → implement. Large changes: verifiable chunks.
- Plans: `docs/plans/YYYY-MM-DD-<feature-name>.md` — temporary, delete after merge. Never commit.
- Act on user instructions directly — no exploratory glob/grep when user names the target.
- Don't verify existence of items already known from plan or memory.

## Plans

### Format
- **PO summary first**: 2-3 sentences max, non-technical, describing what changes for the user/product. Enables fast validation before reading technical detail.
- **Then phases**: phase N depends on N-1 output. Within each phase, mark tasks `[parallel]` or `[seq]`.
- No prose — actionable instructions only. Each task: target files, method signatures/behavior, test criteria.
- All architectural decisions made in plan. No "decide how to…" — Sonnet executes, doesn't architect.

### Sonnet-readiness (per task)
- Target files listed with expected changes
- Input/output or behavior defined
- Test criteria explicit (what to assert)

### Execution model
- Phase with independent tasks → `subagent-driven-development` (parallel Sonnet sub-agents)
- Phase with dependencies → single sequential agent
- Mechanical same-change × N → parallel sub-agents
- One session per phase. Plan encodes execution — no orchestrator needed.

## Quality
- Linters/formatters: before committing only, ONLY on modified files.
- DRY: extract at 3+ occurrences (or 2 if complex). Exception: abstraction obscures intent or coupling > duplication cost.
- Prefer native/library solutions over custom code.

## Token Cost Optimization
- **Parallelize** independent tool calls in one turn. Serialize only when B depends on A's output.
- **Combine Bash** with `&&` when output-independent. Don't combine when you need A's output to decide B, or when failure diagnosis/timeouts differ.
- **No exploratory search when target is known.** Named file/symbol → Read/Grep directly. Skip verifying items known from plan/memory.
- **Direct tools > subagents.** Subagent = full Opus conversation billed on top. Use Grep/Glob/Read for known targets, single-file reads, <3 queries. Use subagents only for open-ended multi-file research or to protect main context.
- **Cheap models on mechanical subagents.** File search, simple refactor, mechanical lookup → `model: "haiku"` or `"sonnet"`.
- **`gh --json field1,field2`** over MCP GitHub tools for simple queries. `minimal_output: true` on MCP list/search calls.

## Coding Standards
- `@Symfony` CS Fixer ruleset + [Symfony standards](https://symfony.com/doc/current/contributing/code/standards.html)
- Backslash-prefix native functions: `\array_map()`, `\sprintf()`, `\count()`
- Prefer `u()` (String component) over native string functions.
- Yoda conditions: `null === $var` not `$var === null`.
- Method order: `__construct` → public → protected → private (`setUp`/`tearDown` first in tests).
- One-line args (except promoted constructors: one per line, trailing comma).
- Existing files: fix only your changes.
- PHPStan level 9 — never ignore/lower.
- Alphabetical: constructor assignments, array keys, YAML keys.
- No magic strings: use constants or enums for domain values reused across files.
- DB queries: repositories only (`src/Repository/`), QueryBuilder only (no DQL). Never inject `EntityManagerInterface` for queries.
- Doctrine migrations: always set `getDescription()` (French, concise).
- **DTOs over arrays**: `readonly` DTO classes in `src/DTO/` or same namespace. `JsonSerializable` only for API/cache.

## Git
- Format: `<type>(scope|branch-name): description` — types: `feat|fix|chore|refactor|docs`
- French descriptions: 3rd-person imperative (`ajoute`, `corrige`, `supprime` — not infinitive).
- Commit title = visible impact, not implementation detail. Technical details in body.
  - `fix`: problem solved. BAD: `utilise PATCH au lieu de PUT` GOOD: `corrige la perte des tomes`
  - `feat`: capability added. BAD: `ajoute CoverSearchService` GOOD: `ajoute la recherche de couvertures`
  - `refactor`/`chore`: improvement. BAD: `extrait getFieldPriority` GOOD: `simplifie la résolution de priorité`
- Never commit `docs/plans/` — always `git reset docs/plans/` before committing.
- Skip `git diff` when you made the edits — diff only to discover changes you didn't make.
- Merges: `--no-ff`

## Translations
- No hardcoded user-facing text — always use translation keys.
- `translations/messages.fr.yaml`: UI labels, titles, buttons, menus.
- `translations/validators.fr.yaml`: `Assert\*` constraint `message:` params.
- Key pattern: `app.<entity>.<context>.<purpose>` (e.g. `app.example.admin.fields.name.label`).
- Twig: `{{ 'key'|trans }}`, with params: `{{ 'key'|trans({'%name%': val}) }}`

## Language
Commits + docs/comments: French. Code identifiers: English. CLAUDE.md: English.

## Recommended Plugins
`php-lsp`, `context7`, `superpowers`, `pr-review-toolkit`, `commit-commands`, `hookify`, `code-simplifier`.

<!-- TEMPLATE:END -->

<!-- PROJECT:START — project-specific content, edit freely.
     Add here: project description, tech stack, architecture, commands, project-specific
     deviations from the template rules.
     If unsure whether a rule belongs here or in TEMPLATE, ask the user before adding. -->

## Project

Claude = sole developer. Keep this file, tests, docs up to date.

Symfony 7.4 + API Platform 4 | React 19 + TypeScript + Vite | MariaDB via DDEV | Tailwind CSS 4 | TanStack Query

## Architecture

```
tarot/
├── backend/    # Symfony API — PHP 8.3
├── frontend/   # React PWA — TypeScript
├── docs/       # user-guide.md, frontend-usage.md, plans/
├── CLAUDE.md
└── CHANGELOG.md
```

> File map (entities, hooks, components, routes, API endpoints): `docs/patterns.md`

## Mandatory TDD

1. Test first (must fail) → 2. Implement (minimum to pass) → 3. Refactor (green tests)

Backend: `make test-back` | Frontend: `make test-front`

## Commands

Always use `make` targets, always prefix with `ddev exec` (hookify-enforced).

| Command | Purpose |
|---|---|
| `ddev exec make dev` | First launch (deps + migrations) |
| `ddev exec make test` | All tests |
| `ddev exec make test-back` | PHPUnit |
| `ddev exec make test-front` | Vitest |
| `ddev exec make lint` | All linters (PHPStan + CS Fixer dry-run + TS) |
| `ddev exec make phpstan` | PHPStan only |
| `ddev exec make cs` | PHP CS Fixer (fix) |
| `ddev exec make db-diff` | Generate migration |
| `ddev exec make db-migrate` | Run migrations |
| `ddev exec make build` | Production build |
| `ddev exec make sf CMD="..."` | Any Symfony console command |

## Git — Project Rules

### Branching

Never push to `main`. Branch + PR only.
- `<type>/<issue-number>-<short-description>` (e.g. `feat/2-entities`)
- One branch = one issue. Branch from `main`, auto-deleted after merge.

### Pull Requests

- PR title: `<type>(scope): description` | Body: summary + `fixes #N`
- Squash merge (`--squash`) → one commit per issue
- Code review (agent) before merge. Update CHANGELOG on branch before merge.

### Releases

Semver `vMAJOR.MINOR.PATCH`. Tag at milestones (not per PR).
Workflow: update CHANGELOG `[Unreleased]` → `[X.Y.Z] - YYYY-MM-DD`, then `git tag vX.Y.Z && git push origin vX.Y.Z`.
GitHub Action auto-creates release from CHANGELOG.

### Issue workflow

```bash
git checkout -b feat/N-description main   # 1. Branch
# TDD, commits referencing #N             # 2. Develop
git push -u origin feat/N-description     # 3. Push
gh pr create --title "..." --body "..."   # 4. PR (fixes #N)
# code review                             # 5. Review
gh pr merge N --squash                    # 6. Merge (auto-delete, auto-close)
```

"Next issue" = highest-priority Todo from board. Full cycle: implement → test → PR → review → merge → CHANGELOG.

## GitHub Project

**Repo**: `Soviann/tarot` | **Project**: `Tarot - Roadmap` (number: 2, owner: Soviann)
Columns: `Backlog` → `Todo` → `In Progress` → `Done`. IDs in MEMORY.md.

Rules:
1. All work starts from an issue. Check existing first.
2. Move to `In Progress` manually (`gh project item-edit`). Other transitions are automatic.
3. New ideas without implementation → `Backlog`.
4. Close via PR `fixes #N`. Labels: use existing only.

## Changelog

`CHANGELOG.md` under `## [Unreleased]`: `### Added|Changed|Fixed|Removed`
Format: `- **Name**: Description`

## Documentation

Maintain: `docs/user-guide.md` (end-user) and `docs/frontend-usage.md` (developer reference).

## Code Conventions — Project-Specific

- PHP backed enums. CS Fixer `@Symfony:risky`.
- API Platform serialization groups (no custom DTOs unless needed).
- **Frontend**: functional components, custom hooks for API, TypeScript strict. UI primitives from `components/ui/`.

<!-- PROJECT:END -->

---
> Source: [Soviann/tarot](https://github.com/Soviann/tarot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
