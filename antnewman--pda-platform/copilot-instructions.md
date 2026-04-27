## pda-platform

> The PDA Platform is a Python monorepo providing AI-powered project delivery assurance for UK government. It exposes 116 MCP tools across 17 modules via a unified server, deployable locally or via Render.

# Claude Code Instructions — PDA Platform

## Project Overview
The PDA Platform is a Python monorepo providing AI-powered project delivery assurance for UK government. It exposes 116 MCP tools across 17 modules via a unified server, deployable locally or via Render.

## Repo Structure
```
packages/
  pda-platform/          Meta-package (pip install pda-platform)
  pm-data-tools/         Core library: parsers, validators, AssuranceStore (SQLite)
  pm-mcp-servers/        17 MCP modules, 116 tools
  agent-task-planning/   AI reliability: confidence extraction, outlier mining
docs/                    Practitioner guides and technical references
.github/workflows/       publish.yml — PyPI publish on version tag
```

## Key Commands
- **Run unified server locally:** `pda-platform-server`
- **Run remote SSE server:** `pda-platform-remote`
- **Run MCP tests:** `cd packages/pm-mcp-servers && python -m pytest`
- **Run API tests:** `cd packages/pm-api && python -m pytest`
- **Run data-tools tests:** `cd packages/pm-data-tools && python -m pytest`

## Working with Claude
1. For ALL git commits, use author `antnewman <antjsnewman@outlook.com>` — NEVER include a "Co-Authored-By: Claude" line
2. Branch workflow: always work on `dev` branch. Never push directly to `main` — that is for PRs/production only
3. Prefer editing existing files over creating new ones
4. After any change to `server.py` or registries, run the MCP test suite to confirm tool counts

## Package Versions — Keep in Sync

All four packages are versioned together. Current version: **1.2.0**

| Package | PyPI | pyproject.toml location |
|---|---|---|
| `pda-platform` | [pypi.org/project/pda-platform](https://pypi.org/project/pda-platform/) | `packages/pda-platform/pyproject.toml` |
| `pm-mcp-servers` | [pypi.org/project/pm-mcp-servers](https://pypi.org/project/pm-mcp-servers/) | `packages/pm-mcp-servers/pyproject.toml` |
| `pm-data-tools` | [pypi.org/project/pm-data-tools](https://pypi.org/project/pm-data-tools/) | `packages/pm-data-tools/pyproject.toml` |
| `agent-task-planning` | [pypi.org/project/agent-task-planning](https://pypi.org/project/agent-task-planning/) | `packages/agent-task-planning/pyproject.toml` |

### When to bump versions

Bump ALL four packages together whenever:
- A new MCP tool or module is added
- A breaking change is made to any public API or store schema
- A significant new feature lands in pm-data-tools or agent-task-planning

Use semantic versioning:
- **Patch** (1.0.x) — bug fixes, doc updates, no new tools
- **Minor** (1.x.0) — new tools added, backward-compatible
- **Major** (x.0.0) — breaking changes to store schema, tool signatures, or APIs

### How to release to PyPI

1. Bump all four `version = "x.y.z"` fields in their respective `pyproject.toml` files
2. Update inter-package dependency pins to `>=` the **previous published version** (NOT the new version — see warning below)
3. Commit: `chore: bump versions to x.y.z`
4. Merge dev → main via PR
5. Tag on main: `git tag vx.y.z && git push antnewman vx.y.z`
6. The GitHub Actions workflow (`.github/workflows/publish.yml`) builds and publishes all four packages automatically
7. After publish completes, update the inter-package pins to `>=x.y.z` in a follow-up commit if needed

### What NOT to do
- Do not publish manually with `twine upload` — use the workflow
- Do not bump only one package — all four must stay in sync
- Do not tag on `dev` — always tag on `main` after the PR is merged
- **Do not pin inter-package deps to the NEW version before it is published** — Render builds from source but pip resolves deps from PyPI. If `pm-data-tools` requires `agent-task-planning>=1.1.0` but 1.1.0 is not yet on PyPI, Render will fail. Keep pins at the last confirmed published version until the new version is live on PyPI.

## MCP Tool Count — Keep in Sync

The tool count (currently **116**) appears in multiple places. When adding new tools, update ALL of these:

| File | What to update |
|---|---|
| `packages/pm-mcp-servers/src/pm_mcp_servers/pda_platform/server.py` | Docstring total, module count in `logger.info` |
| `packages/pm-mcp-servers/tests/test_pda_platform.py` | `test_total_tool_count`, `test_tool_ordering`, expected tool sets |
| `README.md` | MCP modules table, overview paragraph, key features bullet |
| `docs/architecture-overview.md` | Module table |
| `docs/connection-guide.md` | Tool count references |
| `docs/getting-started.md` | Tool count references |
| `packages/pm-mcp-servers/README.md` | Module table, total tool count |
| `packages/pda-platform/README.md` | Module table, total tool count |
| `packages/pda-platform/pyproject.toml` | `description` field tool count |
| `packages/pm-mcp-servers/pyproject.toml` | `description` field tool count and module count |
| `docs/hackathon/CHALLENGE.md` | Numbers section |
| `docs/hackathon/QUICKSTART.md` | What's Available section |
| `server.json` | `description` field (root level) |
| `CLAUDE.md` (this file) | Package versions table, MCP tool count |

## Store Schema — Keep in Sync

When adding a new module that needs persistence:
1. Add `CREATE TABLE IF NOT EXISTS` blocks to `_init_db()` in `packages/pm-data-tools/src/pm_data_tools/db/store.py`
2. Add store methods in the same file (upsert/get pattern)
3. Do this BEFORE launching agents to build the MCP module — agents depend on the store methods existing

## Documentation Requirements — Mandatory for Every Feature

Every new module, tool, or significant capability **must** be accompanied by documentation before it is considered complete. This is non-negotiable.

### For every new MCP module, write:

| Document | Location | What it covers |
|---|---|---|
| For-practitioners guide | `docs/<module-name>-for-practitioners.md` | What the module does, when to use each tool, worked examples with realistic inputs/outputs, common workflows, limitations |
| Section in MCP tools reference | `docs/mcp-tools-reference.md` | Parameter-level reference for each tool — all inputs, outputs, enums, defaults |
| Model card (AI-powered modules only) | `docs/model-cards/<module-name>.md` | Model behaviour, limitations, confidence calibration, failure modes |

### For every new tool added to an existing module:

1. Add a parameter-level entry to `docs/mcp-tools-reference.md`
2. Add an example to the relevant for-practitioners guide
3. If the tool changes what a persona can do, update the relevant persona guide in `docs/guides/`

### Persona-based user guides — keep current

Four guides live in `docs/guides/`. Update them whenever a new module or tool is relevant to that persona's work:

| Guide | Persona | Focus |
|---|---|---|
| `docs/guides/sro-guide.md` | Senior Responsible Owner | Delivery confidence, benefits, escalation, board-ready outputs |
| `docs/guides/pm-guide.md` | Project Manager | Schedule, cost, risk, resources, operational actions |
| `docs/guides/assurance-reviewer-guide.md` | Independent Assurance Reviewer | Gate reviews, IPA methodology, challenge, DCA ratings |
| `docs/guides/portfolio-manager-guide.md` | Portfolio Manager | Cross-project analysis, systemic risk, coherence, intervention |

Each guide must include:
- What tools are most relevant to this persona
- At least 3 worked examples (realistic conversation + tool call sequence + output interpretation)
- How to combine role system prompts with research prompts

### Documentation debt tracking

Current documentation status (update this when docs are written):

| Module | For-practitioners guide | MCP reference | Model card | Persona guide coverage |
|---|---|---|---|---|
| pm-data | ✅ | ✅ v2.0 | n/a | ✅ |
| pm-analyse | ✅ | ✅ v2.0 | ✅ | ✅ |
| pm-validate | ✅ | ✅ v2.0 | n/a | partial |
| pm-nista | ✅ | ✅ v2.0 | n/a | partial |
| pm-assure | ✅ | ✅ v2.0 | ✅ | ✅ |
| pm-brm | ✅ | ✅ v2.0 | ✅ | ✅ |
| pm-gate-readiness | ✅ | ✅ v2.0 | ✅ | ✅ |
| pm-portfolio | ✅ | ✅ v2.0 | n/a | ✅ |
| pm-ev | ✅ | ✅ v2.0 | ✅ | partial |
| pm-synthesis | ✅ | ✅ v2.0 | ✅ | partial |
| pm-risk | ✅ | ✅ v2.0 | n/a | ✅ |
| pm-change | ✅ | ✅ v2.0 | n/a | partial |
| pm-resource | ✅ | ✅ v2.0 | n/a | partial |
| pm-financial | ✅ | ✅ v2.0 | n/a | ✅ |
| pm-knowledge | ✅ | ✅ v2.0 | n/a | ✅ |

## Adding a New MCP Module

Standard pattern (see `pm_brm` as the reference implementation):
1. Add store tables and methods to `store.py` (if needed)
2. Create `packages/pm-mcp-servers/src/pm_mcp_servers/pm_<name>/__init__.py` (empty)
3. Create `server.py` — tool definitions (`<NAME>_TOOLS`) and async handlers
4. Create `registry.py` — re-exports `TOOLS`, `_DISPATCH`, and `dispatch()`
5. Wire into `pda_platform/server.py` — import, add to dispatch loop and `ALL_TOOLS`
6. Update `test_pda_platform.py` — new registry test, tool presence test, updated counts
7. Run `python -m pytest packages/pm-mcp-servers/tests/` — must pass
8. Update all tool count references (see table above) — this includes `packages/pda-platform/README.md`, `packages/pm-mcp-servers/README.md`, both `pyproject.toml` description fields, and `server.json`
9. **Write for-practitioners guide, MCP reference section, and model card (if AI-powered)** — see Documentation Requirements above
10. **Update relevant persona guides with worked examples** — see Documentation Requirements above
11. Bump package versions

## Deployment
- **Render:** `main` branch auto-deploys via `render.yaml`. The build installs from source (not PyPI).
- **PyPI:** Tagged releases only, via GitHub Actions.
- Never push directly to `main` — always PR from `dev`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/antnewman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
