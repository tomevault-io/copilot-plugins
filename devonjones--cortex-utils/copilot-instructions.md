## cortex-utils

> Operational tooling for Cortex: queue management, log aggregation, alerting, and maintenance.

# cortex-utils - Shared Utilities & Operations

Operational tooling for Cortex: queue management, log aggregation, alerting, and maintenance.

## Quick Reference

- **Spec**: `docs/project-spec.md`
- **Repo name**: `cortex-utils` (when created as separate git repo)
- **Main CLI**: `cortex-utils` command

## Key Features

| Feature | Command | Description |
|---------|---------|-------------|
| Partition management | `partitions maintain` | Create/drop daily partitions |
| Queue stats | `queue stats` | Show queue depths and throughput |
| Replay | `queue replay` | Re-enqueue emails by label/date |
| Dead letter | `dead-letter list/retry` | Manage failed jobs |
| Log tailing | `logs tail` | Aggregate container logs |
| Alerter | `alerter run` | Discord exception reporting |
| Health | `health check` | Cross-service health checks |

## Why Partitioned Queue?

Current `queue` table accumulates completed jobs forever:
- DELETE is expensive (bloat, vacuum)
- Table grows unbounded

Partitioned design:
- Daily partitions: `queue_2024_12_13`
- `DROP PARTITION` is O(1), no vacuum needed
- Keep 7 days for debugging/replay
- Failed jobs archived to `dead_letter` before drop

## Environment Variables

```bash
POSTGRES_HOST=10.5.2.21
POSTGRES_DB=cortex
POSTGRES_USER=cortex
POSTGRES_PASSWORD=<secret>
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/xxx/yyy
```

## Development

```bash
uv sync
uv run cortex-utils --help
uv run pytest
```

## Git Workflow

**NEVER push directly to main.** Always:

1. Create a feature branch
2. Make changes and commit
3. Push the branch and create a PR
4. Wait for CI and Gemini Code Assist review
5. Merge via GitHub after approval

This applies to all changes, no matter how small.

## Deployment

- Docker image: `us-central1-docker.pkg.dev/cortex-gmail/cortex/utils:latest`
- Runs on Hades alongside other Cortex services
- Alerter runs continuously; other commands run on-demand or via cron

## Cron Jobs

Via Ofelia on Hades:
- **2 AM daily**: `partitions maintain --retention-days 7`
- **3 AM daily**: `dead-letter purge --older-than 30d`

## Migration

Before using partitioned queue, run one-time migration:
```bash
cortex-utils migrate-queue --dry-run  # Preview
cortex-utils migrate-queue --execute  # Run migration
```

## Related Docs

- [Alerter Spec](../docs/alerter-spec.md) - Discord alerting design
- [Cortex Overview](../docs/cortex-overview.md) - System architecture

---
> Source: [devonjones/cortex-utils](https://github.com/devonjones/cortex-utils) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
