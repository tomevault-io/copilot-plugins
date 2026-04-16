## swarm-reasoning

> **swarm-reasoning** — Multi-agent fact-checking system using structured JSON observations streamed via Redis Streams as inter-agent communication. Ten specialized agents coordinate via swarm reasoning to validate claims against a PolitiFact corpus. Confidence-scored verdicts are produced through three-phase execution: sequential ingestion, parallel fan-out, sequential synthesis.

# CLAUDE.md

## Project Overview

**swarm-reasoning** — Multi-agent fact-checking system using structured JSON observations streamed via Redis Streams as inter-agent communication. Ten specialized agents coordinate via swarm reasoning to validate claims against a PolitiFact corpus. Confidence-scored verdicts are produced through three-phase execution: sequential ingestion, parallel fan-out, sequential synthesis.

## Tech Stack

- **Python 3.11** — Agent logic, orchestration (LangChain), API (FastAPI)
- **TypeScript / Node 20** — CLI tooling, API layer
- **Redis Streams** — Append-only observation log, inter-agent data plane (dev); Kafka graduation path for production (ADR-012)
- **MCP** — Control plane between orchestrator (hub) and agent servers (spokes)
- **Docker Compose** — 13-container local stack
- **Gherkin / Cucumber** — BDD acceptance tests (52 scenarios across 5 feature files)

## Architecture (Key Decisions)

ADRs in `docs/decisions/` as individual MADR v3.0 files (ADR-0001 through ADR-0013).

- **JSON observation schema** — Typed observations with epistemic status (P/F/C/X), published to Redis Streams (ADR-011)
- **Tool-based observation construction** — LLMs never generate raw observations; tools enforce schema validity (ADR-004)
- **Append-only observation log** — Observations are never overwritten; the Redis Streams log is authoritative (ADR-003)
- **Hub-and-spoke MCP** — Orchestrator is the sole MCP client; no subagent-to-subagent connections (ADR-009)
- **Stateless orchestrator** — Reads agent state from Redis Streams on demand; recovers by scanning for STOP messages (ADR-010)
- **Two communication planes** — MCP control plane + Redis Streams data plane, fail independently (ADR-013)
- **Transport abstraction** — `ReasoningStream` interface decouples agents from backend; Redis for dev, Kafka for prod (ADR-012)

## Project Structure

```
docs/
  decisions/               — 13 individual MADR v3.0 ADR files + index + template
  domain/                  — Observation schema spec, OBX code registry (31 codes), claim lifecycle DMN, ERD
  api/openapi.yaml         — 11 REST endpoint paths (Claims, Runs, Verdicts, Audit, System)
  architecture/            — C4 containers, agent topology, MCP topology, DFD, stream lifecycle
  diagrams/sequence/       — 6 Mermaid sequence diagrams (ingestion, fanout, synthesis, etc.)
  diagrams/state/          — Claim lifecycle + observation result status state machines
  features/                — 5 Gherkin files (65 scenarios)
  requirements/nfrs/       — 27 individual NFR files (Planguage + SEI QAS) + index + template
  infrastructure/          — docker-compose.yml, topology diagram
```

## The 10 Agents

Agents communicate via JSON observations published to Redis Streams. Each owns a subset of codes from `docs/domain/obx-code-registry.json`:

1. **ingestion-agent** — Claim intake, entity extraction, check-worthiness gate
2. **claim-detector** — Check-worthiness scoring, claim normalization
3. **entity-extractor** — Named entity recognition (persons, orgs, dates, locations, statistics)
4. **claimreview-matcher** — Google Fact Check Tools API lookup
5. **coverage-left** — Left-leaning source analysis
6. **coverage-center** — Centrist source analysis
7. **coverage-right** — Right-leaning source analysis
8. **domain-evidence** — Domain-specific evidence gathering
9. **blindspot-detector** — Identifies coverage gaps (uses MCP pull pattern)
10. **synthesizer** — Observation resolution, confidence scoring, verdict emission

## Observation Stream Format

Each agent's reasoning session follows: `START → OBS[1..N] → STOP`

- **`status` field** carries epistemic state: `P` (preliminary), `F` (final), `C` (corrected), `X` (cancelled)
- Full schema spec: `docs/domain/observation-schema-spec.md`
- Agent identity is in the `agent` field of each observation
- Stream key format: `reasoning:{runId}:{agent}`

## Code Style

- **Python**: Ruff formatter and linter (configured in devcontainer)
- **TypeScript**: Prettier + ESLint
- Format-on-save is enabled for both languages

## Git Conventions

- **Conventional Commits** format: `<type>(<scope>): <description>`
- Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `ci`, `perf`, `build`, `style`
- Subject line: imperative mood, max 72 chars, no trailing period
- Base commit messages on the **actual diff**, not assumptions or chat context
- **Never** append Claude session URLs, attribution, or AI disclaimers to commit messages
- If multiple logical changes exist, prefer separate atomic commits
- Branch naming: `<type>/<short-description>`

## Environment

- Entry point: `docker compose -f docs/infrastructure/docker-compose.yml up`
- Consumer API: `http://localhost:8000`
- Redis: `redis://localhost:6379` (dev only)
- Required env vars: `ANTHROPIC_API_KEY`, `GOOGLE_FACTCHECK_API_KEY`, `NEWSAPI_KEY`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jzallen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
