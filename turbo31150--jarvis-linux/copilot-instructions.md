## jarvis-linux

> - **Project**: JARVIS OS v15.4 — Multi-LLM orchestrator integrated with Linux

# JARVIS Linux — Project Instructions

## Identity
- **Project**: JARVIS OS v15.4 — Multi-LLM orchestrator integrated with Linux
- **Author**: Turbo (Franc Delmas)
- **Language**: French for docs/comments, English for code
- **Stack**: Python 3.10+, Bash, systemd, Redis, SQLite

## Architecture
```
jarvis-linux/
├── src/jarvis/           # Main Python package
│   ├── cli/              # Unified CLI (jarvis <command>)
│   ├── core/             # Core engine (router, consensus, scheduler)
│   ├── domino/           # Domino chain engine + chains.d/
│   ├── agents/           # Specialized agents
│   ├── integrations/     # External connectors
│   ├── services/         # Service modules (voice, trading, cowork)
│   ├── workers/          # Background workers
│   └── utils/            # Shared utilities
├── infra/                # Infrastructure (no Python code)
│   ├── systemd/          # Service units + timers
│   ├── docker/           # Compose + Dockerfiles
│   ├── scripts/          # Ops scripts by category
│   │   ├── boot/         # Boot sequence
│   │   ├── monitoring/   # Health, GPU, dashboards
│   │   ├── maintenance/  # Backup, cleanup, zombie
│   │   ├── deployment/   # Deploy, install, setup
│   │   └── tools/        # CLI utilities
│   ├── config/           # YAML/JSON configuration
│   ├── cron/             # Scheduled tasks
│   ├── ansible/          # Playbooks
│   ├── sysctl/           # Kernel tuning
│   └── security/         # Firewall, permissions
├── docs/                 # Documentation
│   ├── architecture/     # System design
│   ├── guides/           # User guides
│   ├── reference/        # API, config, system reference
│   ├── operations/       # Runbooks, audits, scans
│   ├── plans/            # Roadmaps, strategies
│   ├── reports/          # Worklogs, session reports
│   └── research/         # Technical research
├── tests/                # Test suite
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── data/                 # Runtime data (ALL gitignored)
│   ├── db/               # SQLite databases
│   ├── logs/             # Log files
│   ├── benchmarks/       # Performance data
│   ├── backups/          # Snapshots
│   └── models/           # ML models
├── assets/               # Static assets (images, branding)
└── .archive/             # Deprecated code (gitignored)
```

## File Routing Rules (MANDATORY)

Every new file MUST go to its correct location. No exceptions.

| File type | Location | Example |
|-----------|----------|---------|
| Python source code | `src/jarvis/<module>/` | `src/jarvis/agents/health.py` |
| CLI commands | `src/jarvis/cli/cmd_*.py` | `src/jarvis/cli/cmd_domino.py` |
| Domino chains | `src/jarvis/domino/chains.d/*.yaml` | `chains.d/service-crash-recovery.yaml` |
| Shell scripts (boot) | `infra/scripts/boot/` | `jarvis-boot-sequence.sh` |
| Shell scripts (monitoring) | `infra/scripts/monitoring/` | `gpu-thermal-guard.sh` |
| Shell scripts (maintenance) | `infra/scripts/maintenance/` | `auto-backup.sh` |
| Shell scripts (deploy) | `infra/scripts/deployment/` | `deploy_omega.sh` |
| Shell scripts (tools) | `infra/scripts/tools/` | `jarvis-cli.py` |
| Systemd units | `infra/systemd/` | `jarvis-master.service` |
| Docker files | `infra/docker/` | `docker-compose.yml` |
| Config YAML/JSON | `infra/config/` | `cluster.yaml` |
| Sysctl tuning | `infra/sysctl/` | `99-jarvis-perf.conf` |
| Cron definitions | `infra/cron/` | `jarvis-crontab` |
| Architecture docs | `docs/architecture/` | `ARCHITECTURE.md` |
| User guides | `docs/guides/` | `QUICKSTART.md` |
| API/config reference | `docs/reference/` | `API_ENDPOINTS.md` |
| Runbooks/audits | `docs/operations/` | `SCAN_GPU.md` |
| Plans/strategies | `docs/plans/` | `IMPROVEMENT_PLAN.md` |
| Reports/worklogs | `docs/reports/` | `WORKLOG.md` |
| Research | `docs/research/` | `ai-consensus-study.md` |
| SQLite databases | `data/db/` | `etoile.db` |
| Log files | `data/logs/` | `domino-engine.log` |
| Benchmark data | `data/benchmarks/` | `20260329_service_crash.bench` |
| Backups | `data/backups/` | `sql_backup_20260329.gz` |
| ML models | `data/models/` | `piper-fr.onnx` |
| Unit tests | `tests/unit/` | `test_consensus_engine.py` |
| Integration tests | `tests/integration/` | `test_api_server.py` |
| E2E tests | `tests/e2e/` | `test_voice_pipeline.py` |
| Deprecated code | `.archive/` | anything removed |

## Coding Conventions
- Python: type hints, docstrings on public functions, Rich for CLI output
- Shell: `set -euo pipefail`, functions over inline, exit codes
- Config: YAML for chains/config, JSON for exports, SQLite for state
- Services: all prefixed `jarvis-*`, must have Restart=on-failure
- Never use `localhost` → always `127.0.0.1`

## Cluster
| Node | Endpoint | Model | Weight |
|------|----------|-------|--------|
| M1 | 192.168.1.85:1234 | qwen3.5-9b | 1.9 |
| OL1 | 127.0.0.1:11434 | deepseek-r1:7b | 1.4 |
| M2 | 192.168.1.26:1234 | deepseek-r1 | 1.5 |
| M3 | 192.168.1.133:1234 | deepseek-r1 | 1.1 |

## GPUs
CUDA_VISIBLE_DEVICES=0,2,3,4,5
- GPU0: RTX 2060 12GB
- GPU2-4: GTX 1660S 6GB (x3)
- GPU5: RTX 3080 10GB

## CLI
```bash
jarvis domino list|trigger|status|bench|deploy|learn
jarvis skill list|audit|improve|merge|dedupe|stats
jarvis cluster health|nodes|route|dispatch
jarvis service status|fix|audit|restart
jarvis deploy cascade|rollback|verify|plan
```

---
> Source: [Turbo31150/jarvis-linux](https://github.com/Turbo31150/jarvis-linux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
