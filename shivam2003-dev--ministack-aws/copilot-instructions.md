## ministack-aws

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Docker Compose environment that runs 40+ AWS services locally via [MiniStack](https://github.com/nahuelnucera/ministack) (LocalStack-based), plus real PostgreSQL and Redis containers for RDS/ElastiCache. No code to build or test — this is purely infrastructure configuration.

## Start / Stop

```bash
# Start all services
docker-compose up -d

# Start only PostgreSQL (standalone, with init script)
docker-compose -f docker-compose-postgres-only.yml up -d

# Stop
docker-compose down

# Stop and wipe all volumes
docker-compose down -v
```

## Verify Services

```bash
# Check how many AWS services are available
curl http://localhost:4566/_localstack/health | jq '.services | keys'

# Run full integration smoke test
bash test-services.sh
```

## Access Points

| Service | Address |
|---------|---------|
| All AWS APIs (MiniStack) | `http://localhost:4566` |
| MinIO Console (S3 UI) | `http://localhost:9001` |
| PostgreSQL | `localhost:15432` — user `admin` / `admin123` |
| Redis | `localhost:16379` |

## AWS CLI Usage

All commands need `--endpoint-url=http://localhost:4566` **or** set env vars once:

```bash
export AWS_ENDPOINT_URL=http://localhost:4566
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
```

Use `export AWS_PAGER=""` to suppress pager output.

## Architecture

```
docker-compose.yml
├── ministack          # nahuelnucera/ministack image — all 40+ AWS services on :4566
├── postgres-rds       # postgres:15 — real DB, accessed via localhost:15432
├── redis-elasticache  # redis:7 — real cache, accessed via localhost:16379
└── minio              # minio/minio — S3 UI at :9001, S3 gateway at :9000
```

MiniStack depends on `postgres-rds` and `redis-elasticache` starting first (enforced via `depends_on`). All containers share `ministack-network`. Data is persisted in named Docker volumes (`ministack-data`, `postgres-data`, `redis-data`, `minio-data`).

The `docker-compose-postgres-only.yml` is a standalone variant that exposes PostgreSQL on the same port (15432) and runs `postgres-init.sql` on first start.

## Troubleshooting

```bash
# Port conflict
lsof -i :4566

# View logs
docker-compose logs ministack
docker-compose logs postgres-rds
docker-compose logs redis-elasticache

# Start in dependency order manually
docker-compose up -d postgres-rds redis-elasticache
docker-compose up -d ministack

# AWS CLI hitting real AWS instead of local?
aws --endpoint-url=http://localhost:4566 s3 ls
```

## Key Files

- `docker-compose.yml` — main environment definition
- `docker-compose-postgres-only.yml` — PostgreSQL-only variant with init script
- `postgres-init.sql` — DB initialization run on first Postgres startup
- `test-services.sh` — smoke test script for all major services
- `AWS_SERVICES_GUIDE.md` — comprehensive usage examples for all 40+ services
- `QUICK_REFERENCE.md` — condensed AWS CLI cheat sheet
- `COMMANDS_WITH_ENDPOINT.md` — all commands with explicit `--endpoint-url` flags

---
> Source: [shivam2003-dev/ministack-aws](https://github.com/shivam2003-dev/ministack-aws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
