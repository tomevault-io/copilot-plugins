## trading-system

> ﻿# AGENTS.md — Governance Command Routing

﻿# AGENTS.md — Governance Command Routing

This file defines the command routing configuration for the `/specify` agent
governance workflow. It maps agent commands to their handler modules.

## Command Routes

| Command              | Handler                                                    |
|----------------------|------------------------------------------------------------|
| `experiment.start`   | `app.governance.experiment_cli:experiment_cli start`       |
| `experiment.pause`   | `app.governance.experiment_cli:experiment_cli pause`       |
| `experiment.resume`  | `app.governance.experiment_cli:experiment_cli resume`      |
| `experiment.complete`| `app.governance.experiment_cli:experiment_cli complete`    |
| `experiment.list`    | `app.governance.experiment_cli:experiment_cli list`        |
| `experiment.show`    | `app.governance.experiment_cli:experiment_cli show`        |
| `experiment.metric`  | `app.governance.experiment_cli:experiment_cli metric`      |
| `experiment.report`  | `app.governance.experiment_cli:experiment_cli report`      |
| `experiment.promote` | `app.governance.experiment_cli:experiment_cli promote`     |
| `experiment.kill`    | `app.governance.experiment_cli:experiment_cli kill`        |
| `experiment.backfill`| `app.governance.experiment_cli:experiment_cli backfill`    |
| `experiment.backfill_pause` | `app.governance.experiment_cli:experiment_cli backfill-pause` |
| `experiment.taxonomy_report` | `app.governance.experiment_cli:experiment_cli taxonomy-report` |
| `experiment.taxonomy_query` | `app.governance.experiment_cli:experiment_cli taxonomy-query` |
| `experiment.governance-report` | `app.governance.experiment_cli:experiment_cli governance-report` |
| `audit.export`       | `app.governance.experiment_cli:experiment_cli audit export`|

## Usage

The `/specify` agent reads this configuration to route governance commands
to the appropriate CLI handlers. Each handler is a Python module path in the
format `module:callable arguments`.

The runtime routing table is also exposed via the API endpoint:

```
GET /api/v1/governance/routes
```

Returns a JSON object containing all registered routes and their count.

## Phase 0 Scope

- Single admin role — all authenticated users have full access.
- Commands are routed to CLI handlers executing in-process.
- Future phases will add role-based dispatch and remote execution.

---
> Source: [Rohithvvs/trading-system](https://github.com/Rohithvvs/trading-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
