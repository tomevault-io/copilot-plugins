## nexgent-open-source-trading-engine

> Conventions for git commit messages and release notes


# Commit & Release Message Conventions

## Commit Messages — Conventional Commits

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification.

### Format

```
<type>(<optional scope>): <short summary>

<optional body — explain WHY, not what>
```

### Types

| Type | When to use |
|------|-------------|
| `feat` | New feature or user-visible capability |
| `fix` | Bug fix |
| `refactor` | Code restructuring with no behavior change |
| `chore` | Build, CI, deps, tooling — no production code change |
| `docs` | Documentation only |
| `test` | Adding or updating tests |
| `perf` | Performance improvement |
| `style` | Formatting, whitespace — no logic change |

### Scopes

Use the package or feature area as scope when the change is focused:

- `backend`, `frontend`, `shared` — package-level
- `agents`, `positions`, `trading-signals`, `wallet` — feature-level
- `api`, `ws`, `db`, `cache` — infrastructure-level

Omit scope for cross-cutting changes that span multiple areas.

### Rules

- **Summary**: imperative mood, lowercase, no period, max 72 chars
- **Body**: wrap at 80 chars, explain *why* the change was made
- **Breaking changes**: add `!` after type/scope and explain in body

```bash
# Good
feat(agents): add DCA re-trigger logic after partial fill
fix(frontend): correct WebSocket reconnect on token refresh
refactor(backend): extract order routing into dedicated service
chore: upgrade pnpm to v9 and refresh lockfile

# Good — breaking change
feat(api)!: rename /agents endpoint to /trading-agents

BREAKING CHANGE: All clients must update the endpoint path.

# Bad
Fixed stuff                   # no type, vague
feat: Update                  # too vague
FEAT(Backend): Add Feature.   # wrong casing, period
```

## Release Messages — Semantic Versioning

Releases use **semver** tags prefixed with `v` (e.g., `v1.2.0`).

| Bump | When |
|------|------|
| **patch** (`v1.0.x`) | Bug fixes, minor improvements, no new features |
| **minor** (`v1.x.0`) | New features, backward-compatible additions |
| **major** (`vX.0.0`) | Breaking API or behavioral changes |

### Release Notes Format

```markdown
## v1.1.0 — <Short Theme>

### Features
- feat(agents): add DCA re-trigger logic (#12)

### Fixes
- fix(frontend): correct balance display after partial fill (#15)

### Other
- chore: upgrade dependencies
```

Group entries by type, reference PR/issue numbers, and keep descriptions concise.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nexgent-ai-org) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
