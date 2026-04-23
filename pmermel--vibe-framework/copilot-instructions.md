## vibe-framework

> Reusable AI-agnostic development framework. Phone is the only required device. Either Claude Code or OpenAI Codex can generate and manage code. GitHub is the canonical system of record.

# Vibe Framework

Reusable AI-agnostic development framework. Phone is the only required device. Either Claude Code or OpenAI Codex can generate and manage code. GitHub is the canonical system of record.

## Provider Neutrality — Read This First

This framework targets both Claude Code and OpenAI Codex equally. Before committing any file, check it against all four gates:

1. **Codex MCP/tool connectivity** — Tool interfaces, action names, and response shapes must be provider-neutral. Would this work if Codex is calling the backend via MCP?
2. **GitHub-centered handoff** — Work state lives in GitHub (issues, branches, PRs, comments), not in any agent's session. Either AI must be able to pick up any branch cold.
3. **Provider-neutral `vibe.yaml`** — No Claude-specific fields, Anthropic-only references, or provider assumptions in the project manifest.
4. **Non-Claude runtime paths** — Every script, workflow, and backend action must function correctly if Claude Code is never involved.

## Canonical Spec

`plan.md` in this directory is the authoritative build specification. Implement from it. `claude-plan.md` is working notes only.

## Intended Usage

Read `.ai/context/USE_CASES.md` alongside `plan.md` before making architecture, workflow, or bootstrap changes. It defines the end-to-end journeys the framework is expected to support in practice.

## Key Decisions

- **Branch model:** `feature/*` → `develop` → `main`
- **Deploy default:** Azure Container Apps (one dedicated environment per project)
- **Deploy static:** Azure Static Web Apps (`adapter: static-web-app` in `vibe.yaml`)
- **Auth:** GitHub Actions OIDC to Azure — no long-lived secrets
- **Repo automation:** GitHub App (not PAT)
- **Workflow refs:** Pinned to release tags or SHAs — never `@main`
- **Project config:** `vibe.yaml` is single source of truth; wrapper workflows generated from `github.workflow_refs`
- **Work queue:** GitHub Issues — AI claims issue, creates linked branch, posts status back
- **Branch ownership:** One active writer at a time; handoff comment required before second agent pushes
- **Bootstrap:** `init.sh` required for first framework bootstrap (backend doesn't exist yet); `bootstrap_framework` backend action is repair/reconfiguration only
- **Promotion:** GitHub-owned — PR merges and environment approval gates only; backend never initiates deployments

## Commands

```bash
# Install dependencies
cd backend && npm ci

# Run tests
cd backend && npm test

# Type-check the backend
cd backend && npx tsc --noEmit

# Validate Bicep templates
az bicep build --file infrastructure/container-apps-env.bicep

# Run init.sh (framework bootstrap) — Phase 3, not yet implemented
# bash scripts/init.sh

# Lint
cd backend && npm run lint
```

## Repository Structure

```
vibe-framework/
├── .ai/context/          # Shared conventions for both agents
├── .devcontainer/        # Codespaces config (Claude Code path)
├── .github/workflows/    # Reusable pipeline logic — all projects call these
├── infrastructure/       # Azure Bicep templates
├── scripts/              # init.sh, setup-azure.sh, setup-github.sh (Phase 3 — not yet implemented)
├── templates/            # Project scaffolds — Phase 3; scaffold logic currently in backend/src/scaffold/
├── AGENTS.md             # Codex operating instructions
├── CLAUDE.md             # This file
├── plan.md               # Authoritative build spec
└── vibe.yaml             # Framework's own manifest
```

## Git Workflow — Non-Negotiable

**Branch before touching any file.** No direct commits to `develop` or `main` — ever.

### Branch naming
| Type | Pattern | Example |
|---|---|---|
| Feature | `feature/<topic>` | `feature/vibe-yaml-schema` |
| Bug fix | `fix/<topic>` | `fix/preview-cleanup-ttl` |
| Docs | `docs/<topic>` | `docs/agents-md` |
| Infrastructure | `infra/<topic>` | `infra/container-apps-bicep` |
| Chore | `chore/<topic>` | `chore/devcontainer-setup` |

### Commit format — Conventional Commits
```
<type>(<scope>): <short description>

# Types: feat, fix, docs, chore, infra, test, refactor
# Examples:
feat(vibe-yaml): add workflow_refs as canonical source for wrapper generation
infra(bicep): add container apps environment template
docs(claude-md): add provider neutrality checklist
chore(devcontainer): install gh, az, and node in codespaces image
```

### Rules
- **One logical change per commit.** If the message needs "and", split it.
- **PR required for all merges.** No exceptions, including docs and chores.
- **Link PRs to issues** when an issue exists.
- **Small, reviewable PRs.** A PR that touches 20+ files across unrelated concerns should be split.

## Testing and Documentation — Required in Every PR

Every PR that adds or changes backend logic must include:

1. **Tests** — vitest unit tests in `backend/src/*.test.ts` covering the changed behaviour.
   - New action: test valid params → 200, missing required params → 400, invalid enum → 400.
   - New handler path: test happy path and each error branch.
   - Run `cd backend && npm test` before pushing. A PR with failing tests will not be merged.

2. **Inline documentation** — JSDoc comment on every exported function explaining what it does, what it does NOT do (e.g. "does not deploy"), and any important constraints.

3. **Context file updates** — if a change affects the backend contract, `vibe.yaml` schema, or bootstrap flow, update the relevant `.ai/context/` file in the same PR.

Tests live alongside the code they test: `src/foo.ts` → `src/foo.test.ts`.

## File Generation Checklist

After writing any file, verify:
- [ ] Works if Codex is the active agent (not Claude)
- [ ] `vibe.yaml` contains no provider-specific fields
- [ ] Bootstrap path functions without Codespaces
- [ ] GitHub state (not session memory) is the handoff mechanism

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pmermel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
