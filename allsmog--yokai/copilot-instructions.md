## yokai

> Supply chain tripwires. Catch dependency attacks before they land.

# Yokai (妖怪) — Supply Chain Deception Platform

Supply chain tripwires. Catch dependency attacks before they land.

## Project Structure

- Source: `src/` — CLI entry in `src/cli.ts`, core types in `src/types.ts`, config in `src/config.ts`
- Agent runtime: `src/agent-runtime/` — pi-ai LLM abstraction
- Bus: `src/bus/` — in-process event bus with typed EventEnvelopes
- DAG: `src/dag/` — task registry, topological sort, orchestrator with checkpointing
- Store: `src/store/` — SQLite persistence (canary tokens, alerts, interactions, checkpoints)
- Registries:
  - `src/registries/npm/` — npm protocol canary registry
  - `src/registries/pypi/` — PyPI Simple Repository API canary registry
  - `src/registries/maven/` — Maven repository protocol canary registry
  - `src/registries/go/` — Go module proxy (GOPROXY) canary registry
  - `src/registries/proxy/` — Transparent registry proxy (passthrough + interception)
- Canary:
  - `src/canary/token.ts` — canary token generation
  - `src/canary/package-builder.ts` — package building (plain + LLM-assisted)
  - `src/canary/pipeline-tokens.ts` — CI/CD config generation (.npmrc, pip.conf, settings.xml, .yarnrc.yml, GOPROXY)
  - `src/canary/git-decoy.ts` — fake Git smart HTTP endpoints
- Discovery: `src/discovery/` — namespace scanning, .npmrc parsing, public registry checks
- Detection: `src/detection/` — alert classification, severity scoring, IP enrichment
- Typosquat: `src/typosquat/` — algorithmic + LLM variant generation, continuous monitoring
- Report: `src/report/` — JSON and SARIF report generation with MITRE ATT&CK mappings
- Integrations: `src/integrations/` — Slack, Teams, PagerDuty, generic webhook dispatch
- Stages: `src/stages/` — 5-stage pipeline (S1→S5)
- Deployment: `Dockerfile`, `docker-compose.yml`, `.github/workflows/yokai-action.yml`

## Build, Test, and Development Commands

- Runtime: Node 22+
- Package manager: pnpm
- Install deps: `pnpm install`
- Dev run: `pnpm dev` or `npx tsx src/cli.ts`
- Build: `pnpm build`
- Type-check: `pnpm typecheck`
- Tests: `pnpm test`
- Single test file: `npx vitest run tests/alert-engine.test.ts`
- Docker: `docker compose up` (mounts repo at /repo, persists SQLite in volume)

## Architecture: 5-Stage Pipeline DAG

```
S1 Discover Namespaces → S2 Generate Canaries → S3 Deploy Registries → S4 Configure Monitoring → S5 Baseline Traffic
```

Each stage is a `YokaiTask<TIn, TOut>` with structured output checkpointed to SQLite.

## Key Conventions

- Model spec format: `provider:modelId` (e.g., `anthropic:Codex-sonnet-4-6`)
- All LLM calls go through PiAiAgentRuntime
- Stage outputs are checkpointed to SQLite after completion
- `yokai resume <runId>` skips completed stages
- Alerts are classified with MITRE ATT&CK technique IDs
- Alert types: dependency-confusion, credential-probe, unauthorized-publish, canary-download, namespace-probe, typosquat-claim, config-tamper
- Registry protocols: npm, PyPI, Maven, Go (all via Hono)
- Transparent proxy mode: forwards to upstream, intercepts monitored names
- Git decoy: smart HTTP protocol endpoints that log clone/fetch/push attempts
- Canary tokens phone home via HTTP POST with system metadata
- Webhook dispatch: Slack Block Kit, Teams MessageCard, PagerDuty Events v2, Generic with HMAC signing
- Deploy CLI flags: `--mode proxy`, `--protocol pypi`, `--webhook-slack <url>`

## CLI Commands

- `yokai scan --repo <path>` — discover internal package namespaces
- `yokai deploy --repo <path> --port 4873` — deploy canary registry (npm/pypi/maven/go)
- `yokai deploy --mode proxy --upstream https://registry.npmjs.org` — transparent proxy mode
- `yokai deploy --webhook-slack <url>` — dispatch alerts to Slack
- `yokai monitor` — show live alerts from the latest run
- `yokai report --format sarif` — generate SARIF report from alerts
- `yokai resume <runId>` — resume interrupted run from checkpoint
- `yokai typosquat --repo <path>` — scan public npm for claimed typosquat variants
- `yokai canary-configs --registry-url <url> --scope @myorg -o ./canaries` — generate CI/CD canary configs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/allsmog)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/allsmog)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
