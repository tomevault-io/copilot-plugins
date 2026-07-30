## 30-quality-gates

> Quality and operational gates for data engineering work


# Quality Gates

For production-facing data work:

- use `data-quality-and-contract-testing`
- use `data-observability-and-sla-management`
- use `incident-triage-and-pipeline-recovery` for incident work
- for `/backfill` or replay work, require `safe-backfill-and-replay-orchestration` and `templates/backfill-plan.yaml` evidence
- for Kafka or serverless Spark changes, load `kafka-resilience-and-schema-evolution` or `spark-serverless-reliability-and-state-management` as appropriate
- require evidence for correctness, replay safety, and publish readiness

---
> Source: [vaquarkhan/data-engineering-agent-skills](https://github.com/vaquarkhan/data-engineering-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
