## zzz

> > Domain context (DAS physics, key concepts, data flow, deployment topology):

# SequoIA — Claude Code Project Instructions

> Domain context (DAS physics, key concepts, data flow, deployment topology):
> see [`docs/DAS-PRIMER.md`](docs/DAS-PRIMER.md)

## Tempo and session length

- **Never defer work to "tomorrow" or "later" because the session feels long.** A session that's been running for hours is not a reason to wrap up, checkpoint, call it a day, or push items to another time. Sessions have no time limit that matters to the user. If there's work to do and you're able to do it, keep doing it.
- **Never propose "let's pick this up tomorrow / in a follow-up session / after a break" as a way to manage scope.** If you think the scope is too large for one PR, split it into multiple PRs today. If you think something needs review, open it now and keep working on the next thing. Scope management is about *splitting*, not *deferring*.
- **Never suggest waiting for "morning" or "after lunch" or any wall-clock excuse.** The user's calendar is their concern, not yours. Assume the user is present and wants to keep working unless they explicitly say otherwise.
- **Never say "we've done a lot today, want to stop here?"** Don't editorialize about session length. Just keep moving through the plan.
- This rule is load-bearing: it exists because the user has been driven up the wall by implicit deferral. Follow the letter and the spirit.

## Workflow

### Issue-Driven Development

Work is tracked through GitHub issues. Before starting any task:

1. **Find or create the GitHub issue.** One issue per concern. Label it (`bug`, `enhancement`, `refactor`, `tech-debt`, `infrastructure`, `performance`). Use the standard format from `.github/ISSUE_TEMPLATE/`.
2. **`TODO.md` is the sprint board** — it lists only the current sprint priorities. The backlog lives in GitHub Issues. Do not duplicate backlog items in TODO.md.
3. **Branch names reference the issue:** `feat/42-flow-switching`, `fix/15-token-refresh`.

### Task Pattern

1. **Create a branch** from main: `feat/N-description`, `fix/N-description`, `refactor/N-description`, `chore/N-description`, or `perf/N-description` (where N is the issue number)
2. **Write tests first** when adding features or fixing bugs
3. **Implement** the change
4. **Validate**: run `make lint && make typecheck`
5. **Commit** with conventional message: `feat:`, `fix:`, `refactor:`, `chore:`, `docs:`, `test:`. **Never add `Co-Authored-By` trailers** — commits should look like they come from the developer alone. **Never add "Generated with Claude Code" footers** to PR descriptions.
6. **Push**: `git push -u origin <branch>`
7. **Open a PR**: `gh pr create --title "short title" --body "Closes #N\n\n## Summary\n- what changed\n- why"` — the `Closes #N` line is **mandatory** so GitHub auto-closes the issue on merge.
8. **Never merge** — the human reviews and merges PRs
9. **Address PR feedback** — when asked to fix PR comments, read them with
   `gh api repos/Traquolix/Zzz/pulls/<number>/comments` and
   `gh pr view <number> --comments`, then fix, commit, and push

If the user doesn't specify a branch name, ask for one. Never work directly on main.

### PR Hygiene

- **One concern per PR.** If you notice an unrelated issue while working, create a separate issue and branch for it. Never bundle unrelated fixes.
- **Keep PRs reviewable.** If the diff exceeds ~500 lines of logic changes, consider splitting into smaller PRs. Formatting-only changes (reindentation, import reordering) should be a separate commit so reviewers can skip them.
- **Always include `Closes #N`** (or `Fixes #N`) in the PR body so the issue auto-closes on merge. If the PR doesn't fully resolve an issue, use `Relates to #N` instead.
- **No cross-contamination between sim and live paths.** Every REST endpoint and WebSocket handler that can serve both simulation and live data must be flow-aware. Never fall back from one data source to the other without checking the user's active flow.
- **Use existing patterns.** Before writing inline error handling, check if a decorator or helper already exists (e.g., `@clickhouse_fallback`). Before writing inline broadcasts, use the shared broadcast helpers (`broadcast_per_org`, `broadcast_to_orgs`, `broadcast_shm` from `apps.realtime.broadcast`). Consistency matters more than local convenience.

### After Merge — Deploying

#### Preprod (automatic on merge)

Merging any PR to `main` auto-deploys to preprod via `.github/workflows/deploy.yml`
(with auto-rollback on health check failure — see #498, #519, the producer
recovery watchdog in #535, and the rollback drill in #521). There's nothing to do
manually on the preprod side after a merge; the deploy workflow takes over.

Watch the deploy at: https://github.com/Traquolix/Zzz/actions/workflows/deploy.yml

The preprod frontend stack (NPM + frontend nginx + Authentik) is also auto-synced
from the repo on every merge — see `deploy-preprod-frontend-stack` job (#473).

#### Prod (manual, via `make release`)

Since #510/#511, prod deploys are **manual-only** via `workflow_dispatch`. The
manual trigger IS the gate — the `prod` GitHub Environment no longer has a
required-reviewer rule (redundant with the manual trigger). The manual flow
prevents accidental prod deploys while keeping releases cheap and explicit.

The release happy path is:

```bash
# 1. Check what you're about to ship
make release-status
# Shows preprod SHA (what will be deployed) and current prod SHA

# 2. Ship it
make release
# Interactive confirmation: shows SHAs, waits for y/N, triggers workflow
```

`make release` runs `gh workflow run deploy.yml -f target=prod` under the hood,
with the current preprod SHA as the default target. The workflow's backend job
resolves the target SHA by reading `https://test.sequoia-analytics.tech/api/health`
and deploys whatever preprod is currently running.

Watch the deploy at: https://github.com/Traquolix/Zzz/actions/workflows/deploy.yml

#### Rollback

If prod is broken and you need to roll back to a previous SHA:

```bash
# 1. Find the rollback target
git log origin/main --oneline | head -20

# 2. Trigger the rollback (interactive confirmation)
make release-rollback SHA=<first-7-or-more-chars>
```

`make release-rollback SHA=<target>` runs
`gh workflow run deploy.yml -f target=prod -f image_tag=<sha>`, which deploys the
specified SHA explicitly instead of reading preprod's current state. The normal
deploy path runs (including the digest resolution from #498, the version file
write from #519, the health check, and the auto-rollback if the rollback deploy
itself fails).

For the full drill procedure and what to expect from an auto-rollback, see
`docs/runbooks/deploy-rollback-drill.md` (shipped as part of #521).

#### What "auto-rollback on health check failure" actually does

The preprod and prod deploy jobs both have a `Rollback on failure` step that
fires when the health check step times out (containers never reach `healthy`
state within 180s). The rollback:

1. `git reset --hard $PREVIOUS_SHA` on the server's `/opt/Sequoia` checkout
2. Unsets the failed deploy's `BACKEND_IMAGE` / `PROCESSOR_IMAGE` / `AI_ENGINE_IMAGE` env vars (otherwise they'd point at the broken deploy's digests via `$GITHUB_ENV`)
3. Writes the previous SHA to `.version.json` (the bind-mounted file the backend reads at startup, #519)
4. Pulls the previous images from GHCR
5. Re-resolves digests for the rollback target
6. Runs `docker compose up -d` with the rollback env vars set
7. Prod rollback also exits the workflow with code 1 so the Actions run is flagged red

**The auto-rollback has been tested via the #521 drill.** If you're doing a
release and something goes wrong, it's designed to catch you — but if you want
a faster manual rollback, `make release-rollback SHA=<prev>` is the quickest
way to explicit-pin prod to a specific working SHA.

#### Manual SSH fallback

If GitHub Actions itself is broken and neither `make release` nor
`make release-rollback` can work, there's a manual fallback script:

```bash
./scripts/deploy-manual.sh         # (manual SSH-based deploy, #524)
```

This is the last-resort path. Use it only when the normal workflow is
unavailable — `make release` is the primary path.

## Validation

**Run before considering any task complete:**

```bash
make lint && make typecheck
```

If any step fails, fix the issue and re-run. Do not report completion until all pass.

### AI Engine Tests

```bash
make test              # Run AI engine test suite (234 tests)
```

Tests run in CI on every PR **on GPU** (`CUDA_VISIBLE_DEVICES=0`, RTX 4000).
The suite includes golden snapshot tests that compare inference output against
a saved reference from real DAS data.

**When you update the DTAN model weights, detection thresholds, or preprocessing:**

Golden snapshots must be re-recorded **on the production server** (GPU) because
CI runs tests on GPU. Snapshots recorded on CPU won't match GPU output due to
floating-point differences. Both recording and testing run with `torch._dynamo`
disabled (eager mode) to ensure bit-identical results across runs.

```bash
# 1. Push your code change to a branch
# 2. On the server, checkout the branch and re-record:
ssh beaujoin@192.168.99.113
cd /home/beaujoin/actions-runner/_work/Zzz/Zzz
git fetch origin <branch> && git checkout <branch>
CUDA_VISIBLE_DEVICES=0 make snapshot-confirm

# 3. Copy the updated snapshots back to your local machine:
scp beaujoin@192.168.99.113:/home/beaujoin/actions-runner/_work/Zzz/Zzz/services/pipeline/tests/ai_engine/fixtures/golden_*_x86_64.npz \
    services/pipeline/tests/ai_engine/fixtures/

# 4. Commit the updated .npz files into your PR
git add services/pipeline/tests/ai_engine/fixtures/*.npz
git commit -m "test: update AI engine golden snapshots"
```

If you skip this, the snapshot tests will fail in CI. `make snapshot-confirm`
uses `--rerun` (re-records from existing `golden_input.npz`, no HDF5 needed).
For full regeneration from raw HDF5 source data, use `make snapshot-regenerate`.

## Makefile — Always Use It

The Makefile is the single entry point for all dev operations. **Never run `python3`,
`ruff`, `mypy`, or `pip` directly** — always use the Makefile targets, which use
service-local venvs automatically.

| Task | Command |
|------|---------|
| First-time setup (venvs + deps) | `make setup` |
| Start dev servers | `make dev` |
| Lint all code | `make lint` |
| Type-check all code | `make typecheck` |
| Auto-format | `make format` |
| Security scan | `make security` |
| Run tests | `make test` |
| Update AI golden snapshots | `make snapshot-confirm` |
| Run AI benchmarks | `make bench` |
| Save benchmark baseline | `make bench-save` |
| Full CI pipeline | `make ci` |
| Docker stack up/down | `make up` / `make down` |
| Rebuild one service | `make rebuild SERVICE=<name>` |
| View logs | `make logs SERVICE=<name>` |
| Check release status | `make release-status` |
| Ship current preprod to prod | `make release` |
| Roll back prod to a specific SHA | `make release-rollback SHA=<target>` |
| Manual backup | `make backup` |
| Restore from backup | `make restore BACKUP=--latest` |

If a venv is missing, run `make setup` first. The `make dev` target auto-creates the
backend venv on first run, but `make lint` and `make typecheck` expect venvs to exist.

## Conventions

### Python
- Formatter/linter: `ruff` only (NOT black). Line length 100.
- Python 3.10 everywhere (pipeline and backend). Must match Docker images.
- Type hints required on all new code.
- Import order: stdlib → third-party → local (enforced by ruff isort).
- Logging: `logging.getLogger(__name__)`. Never `print()`.
- Error handling: raise specific exceptions, never bare `except:`.
- Commit messages: conventional commits (`feat:`, `fix:`, `refactor:`, `chore:`, `docs:`, `test:`).

### TypeScript (frontend)
- Strict TypeScript with ESLint.
- State management: Zustand stores for client state; React Query (`@tanstack/react-query`) for server state (data fetching, caching, background refetch).
- Styling: Tailwind CSS v4.
- Components: shadcn/ui base components.
- i18n: all user-visible strings in `src/i18n/en.json` and `src/i18n/fr.json`.

### Kafka
- All messages use Avro serialization with Schema Registry.
- Avro schemas must be backwards-compatible — add optional fields with defaults, never remove or rename fields.

### Backend
- Django views: always set `permission_classes` (`IsActiveUser`, `IsAdminOrSuperuser`, or `IsSuperuser`).
- All data queries are org-scoped — filter by `request.user.organization`. Never return unscoped data.
- ClickHouse queries: use `apps.shared.clickhouse.query()` — never create clients directly.
- URL routes: add to `apps/api/urls.py`.

### Testing
- Pipeline: `pytest` with `pytest-asyncio`. Fixtures in `tests/conftest.py`. Integration tests in `tests/integration/` (require Docker stack).
- Backend: `pytest-django` with `DJANGO_SETTINGS_MODULE=sequoia.settings.test`. Factory Boy factories in `tests/factories.py`. Key fixtures: `org`, `admin_user`, `authenticated_client`, `mock_clickhouse_query`.
- Frontend: `vitest` with `@testing-library/react`. Test files colocated: `Component.test.tsx` next to `Component.tsx`.

## Prohibitions

1. **Never modify files in `tests/integration/`** — integration tests are the contract.
2. **Never use `eval()`, `pickle.loads()`, `exec()`, or `subprocess(shell=True)`**.
3. **Never hardcode secrets** (passwords, tokens, API keys). Use environment variables.
4. **Never import across service boundaries** — `pipeline/` must not import from `platform/` and vice versa. Within pipeline, `processor/` and `ai_engine/` import from `shared/` and `config/` only, never from each other.
5. **Never add dependencies** without updating `pyproject.toml` (pipeline), `requirements.txt` (backend), or `package.json` (frontend).
6. **Never push directly to main** — always work on feature branches.
7. **Never skip validation** — always run `make lint && make typecheck`.
8. **Never commit `.env` files, secrets, or credentials**.
9. **Never modify model weight files** (`.pth`, `.pt`) — those are training outputs.
10. **Never modify Avro schemas** without considering backwards compatibility.
11. **Never create a Django view without `permission_classes`**.
12. **Never query ClickHouse or PostgreSQL without org-scoping** in the backend.

## Architectural Invariants

1. **One Kafka partition per fiber** — strict message ordering required. The sliding-window buffers in Processor and AI Engine produce wrong results with out-of-order messages.
2. **Single-instance multi-fiber** — one Processor and one AI Engine handle all fibers. The Processor subscribes to `das.raw.*` (topic pattern). The AI Engine dispatches per-fiber batches independently with GPU lock serialization.
3. **Config hot-reload** — `FiberConfigManager` watches `fibers.yaml` for changes. Never cache fiber config in module-level variables.
4. **Multi-tenant backend** — every API query is scoped to `request.user.organization`. Superuser admin endpoints are the only exception.
5. **ClickHouse 3-tier storage** — `detection_hires` (48h TTL) → `detection_1m` (90d) → `detection_1h` (forever). Aggregation is handled by ClickHouse materialized views, not Python.
6. **ServiceBase pattern** — all pipeline services inherit from `shared.service_base.ServiceBase`.
7. **Transformer hierarchy** — Consumer → Producer → Transformer → MultiTransformer → BufferedTransformer → RollingBufferedTransformer. Choose the right level for new services.

## PR Reviews

When asked to review a PR, follow the review format and checklist in `tools/prompts/pr-review-agent.md`.
That prompt covers hard rules (blocking), architectural invariants, code quality, and performance concerns
tailored to this project.

## Key Files

| File | Purpose |
|------|---------|
| `Makefile` | All dev operations — lint, typecheck, setup, dev servers, Docker |
| `docker-compose.infra.yml` | Shared infrastructure (Kafka, ClickHouse, PostgreSQL, Redis, otel) |
| `docker-compose.services.yml` | Application services (backend, processor, ai-engine) |
| `docs/DAS-PRIMER.md` | DAS physics, domain concepts, data flow, deployment topology |
| `scripts/README.md` | Index of all operational shell scripts and when to use them |
| `scripts/setup-server.sh` | Bootstrap a new server (Docker, GPU toolkit, GH runner, backups, nginx) |
| `scripts/setup-proxy.sh` | Bootstrap the frontend server's NPM + nginx containers |
| `scripts/setup-authentik.sh` | Deploy Authentik SSO to the frontend server |
| `scripts/backup.sh` | Nightly DB backup with cron self-install |
| `scripts/restore.sh` | Restore from backup |
| `scripts/deploy-manual.sh` | Manual deploy fallback (SSH-based, only when GH Actions is broken) |
| `docs/ROLLBACK.md` | Rollback procedures |
| `TODO.md` | Current sprint priorities |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Traquolix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
