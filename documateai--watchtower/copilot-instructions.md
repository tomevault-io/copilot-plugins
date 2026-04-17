## watchtower

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Watchtower is a cross-platform Laravel package for queue monitoring and worker management. It's a Laravel Horizon alternative that uses polling instead of PCNTL signals, enabling full Windows support. Published as `documateai/watchtower` on Packagist.

**Requirements:** PHP 8.2+, Laravel 11/12, Symfony Process. Redis optional (control channel supports Redis or Database drivers).

## Commands

```bash
# Install dependencies
composer install

# Run tests
composer test

# Format code (Laravel Pint, PSR-12)
composer format

# Frontend (optional, dashboard currently uses Blade + Alpine.js from CDN)
npm run dev       # Vite dev server
npm run build     # Build production assets
```

No test files exist yet. Test infrastructure is set up with PHPUnit 10/11 and Orchestra Testbench 9/10.

## Architecture

### Core Control Flow

```
Supervisor (long-running) → spawns Worker processes via Symfony Process
Workers poll CommandBus every 3s for commands (stop/pause/resume/restart/terminate)
Dashboard → Controllers → WorkerManager → writes commands via CommandBus
JobMonitor listens to Laravel queue events → records to watchtower_jobs table
```

The key design choice: **no PCNTL signals**. All worker control uses a `CommandBusInterface` with key-value semantics (`watchtower:worker:{id}:command`). Workers poll, read, execute, and clear these keys. Supervisor termination uses `watchtower:terminate`. The command bus has two drivers: `redis` (default) and `database`, configured via `watchtower.command_bus`.

### Namespace: `Documateai\Watchtower`

### Services (registered as singletons)

- **CommandBusInterface** (`src/Contracts/CommandBusInterface.php`) - Abstraction for worker control commands (`put`/`get`/`forget`). Implementations: `RedisCommandBus` and `DatabaseCommandBus` in `src/Services/CommandBus/`. Driver selected via `watchtower.command_bus` config.
- **WorkerManager** (`src/Services/WorkerManager.php`) - Spawns/stops workers, sends commands via CommandBus, discovers queues from 6 sources (Redis keys, database jobs table, failed_jobs, watchtower_jobs, active workers, queue config). Cross-platform PID checking via `tasklist` (Windows) / `posix_kill` (Unix).
- **JobMonitor** (`src/Services/JobMonitor.php`) - Fail-silent event listeners for JobQueued/Processing/Processed/Failed/RetryRequested. Records job lifecycle to database.
- **MetricsCollector** (`src/Services/MetricsCollector.php`) - Aggregates throughput, queue depth, average duration stats.

### Commands

| Command | Purpose |
|---|---|
| `watchtower:supervisor` | Main long-running process, manages worker pool per config |
| `watchtower:worker {queue}` | Individual worker, wraps Laravel's queue worker with polling control |
| `watchtower:restart` | Zero-downtime restart via command bus signal |
| `watchtower:terminate` | Graceful shutdown of supervisor + all workers |
| `watchtower:prune` | Clean up old job records per retention config |

### Worker Balancing

- **simple** (default): Each worker processes ALL queues (comma-separated)
- **auto**: Round-robin assignment, each worker gets one queue

### Dashboard

Blade views + Alpine.js at `/{watchtower.path}` (default: `/watchtower`). Gate-based auth (`viewWatchtower`) - local-only by default. Controllers in `src/Http/Controllers/`: Dashboard, Jobs, FailedJobs, Workers, Metrics.

### Database

Three tables: `watchtower_jobs` (job tracking with status/payload/exceptions/timing), `watchtower_workers` (PID, status, heartbeats), and `watchtower_commands` (command bus key-value store, only used when `command_bus` is `database`). Configurable connection via `watchtower.database_connection`.

## Conventions

- Commit messages: `type(scope): description` (e.g., `feat(dashboard): add queue depth chart`)
- Branch names: `feature/`, `fix/`, `docs/`, `refactor/`
- All services are fail-silent (wrapped in try/catch) to never break the host application's queue system
- Config values are env-driven with sensible defaults (see `config/watchtower.php`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/documateai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
