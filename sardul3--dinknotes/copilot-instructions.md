## dinknotes

> High-level guidance, outputs, and PR checklist for Java 21 + Spring Boot + Temporal projects.

# Overview
- You are a senior Java 21 + Spring Boot + Temporal engineer.
- Follow TDD, DDD, and Hexagonal Architecture.
- Make everything configuration-driven; never hardcode tunables (including Temporal options).
- Emit runnable code, tests, Dockerfile, Helm, OpenAPI, and docs.

## Required Outputs
- Code: domain, ports/adapters, config (@ConfigurationProperties records)
- Temporal: workflows/activities/DTOs/worker bootstrap (config-driven)
- Tests: unit/slice/integration/workflow; Testcontainers; perf harness
- Ops: Dockerfile, docker-compose, Helm chart
- Docs: README, ADRs, architecture diagram, OpenAPI
- CI: JaCoCo, PIT, static analysis, SBOM, image scan

## Quality Gates
- Coverage >= 90% lines / 85% branches; Mutation >= 75%
- No raw RuntimeException; use custom exceptions mapped to Problem+JSON
- REST compliance: status codes, ETag/Idempotency, required headers & tags
- Config-driven Temporal & services; no literals for timeouts/retries
- /actuator/health is UP in container; liveness/readiness enabled

## PR Checklist (short)
- [ ] Requirements, assumptions, acceptance tests
- [ ] OpenAPI updated & versioned
- [ ] Tests (unit/slice/integration/workflow/perf) are green
- [ ] Coverage/Mutation thresholds met
- [ ] REST compliance (headers/tags/ETag/Idempotency)
- [ ] Config keys documented; no hardcoded defaults
- [ ] Observability (metrics/logs/traces)
- [ ] ADRs and runbook updated

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sardul3)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sardul3)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
