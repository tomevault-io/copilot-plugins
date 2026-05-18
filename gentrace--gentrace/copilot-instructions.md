## gentrace

> > Important: before finishing a task, consider whether this agents.md should be updated with helpful information you learned while completing the task. If yes, please suggest an edit.

> Important: before finishing a task, consider whether this agents.md should be updated with helpful information you learned while completing the task. If yes, please suggest an edit.

# What is Gentrace
Gentrace is an LLM agent tracing and evaluation tool.

# Navigating the codebase
This is a Typescript codebase. Most of the code lives in the `app` dir.

# Making changes
Any time you make changes in the `app` dir, from the `app` dir use:
- `yarn lf` to lint staged changes.
- `yarn jest -i <path from app to test>` to run tests. Don't run all tests - it will take too long.
- `yarn jest:reset -i <path from app to test>` to run tests if you made a migration, as you will need to apply the migration to the test db.
- `yarn tsc-inc` to run TypeScript's incremental compiler for quick build feedback. It has been warmed up for you.

The test environment has access to databases. When writing tests, it's preferable to use the `useSeedTestDb` hook and operate on a db instance that gets cleaned up and repopulated for you rather than mocking database calls. In general, use mocking sparingly, eg for true network calls.

# Readme
Read the README.md file in the root of the repo for an overview of the project.
The README.md file is a good place to start when you're not sure what to do.
The README has instructions for running the app, database migrations, tests, and more.
If anything is unclear, ask me and suggest an update to the README.






README.md copied from the app dir:

# Gentrace application

## Prerequisites

### Set up local services

All required services (PostgreSQL, ClickHouse, Kafka, etc.) are managed via
Docker Compose.

See the [`local` README](../local/README.md) for setup instructions.

### Environment configuration

Add this line to your `app/.env` to point to your PostgreSQL instance:

```sh
DATABASE_URL=postgresql://gentrace:gentrace123@localhost:5432/gentrace
```

## Installation

```sh
yarn install
```

## Run database migration

```sh
yarn migrate-ch
yarn migrate-dev
```

## Run the development server

```sh
yarn dev
```

## Unit tests

```sh
yarn jest
```

To run a specific test and rerun when you make changes:

```sh
yarn jest tests/batchEvaluation.test.ts --watch
```

## Resetting

Reseed local Postgres DB + Clickhouse DB:

```sh
cd ../app
yarn db-seed-reset
yarn migrate-ch
yarn seed-ch
```

## Impersonation

See the [impersonation README](scripts/prod-snapshot/README.md) for details.







README.md copied from the local dir:





# Local Development Environment

This directory contains the local development setup for Gentrace using Docker Compose.

**⚠️ Important: Docker is required for this local development environment. All services run in Docker containers.**

## Prerequisites

1. **Docker and Docker Compose installed on your system** (Required)
2. Copy `.env.example` to `.env` and configure your environment variables

## Quick Start

1. Copy the environment file:

   ```bash
   cp .env.example .env
   ```

2. Install dependencies:

   ```bash
   yarn install
   ```

3. Start the services:

   ```bash
   yarn start
   ```

4. Stop the services:

   ```bash
   yarn stop
   ```

5. Restart the services:

   ```bash
   yarn restart
   ```

   To completely remove all volumes and start from scratch:

   ```bash
   docker-compose down -v
   ```

## Services

The Docker Compose setup includes:

- **PostgreSQL** (port 5432): Main database
- **ClickHouse** (ports 8123, 9000): Analytics database
- **Kafka** (ports 9092, 29092): Message streaming
- **Zookeeper** (port 22181): Kafka coordination
- **Kafka Connect** (port 8083): Data integration
- **Kafka UI** (port 8080): Kafka management interface

## Configuration

### Environment Variables

See `.env.example` for all available configuration options.

### Network

All services run on a custom Docker network (`gentrace`) with static IP
addresses for consistent connectivity.

## Troubleshooting

### Docker Space Issues

If you encounter issues with data directory creation, you may need to clear
Docker builder space:

```bash
docker builder prune
```

### Connecting to Services

- **ClickHouse**: Access via HTTP on `http://localhost:8123` or native protocol
  on port 9000
- **PostgreSQL**: Connect using
  `postgresql://gentrace:gentrace123@localhost:5432/gentrace`
- **Kafka UI**: Access the web interface at `http://localhost:8080`

### Data Persistence

Data is persisted in Docker named volumes:

- `postgres-data`: PostgreSQL data
- `clickhouse-data`: ClickHouse data
- `clickhouse-logs`: ClickHouse logs
- `kafka-data`: Kafka data
- `zookeeper-data`: Zookeeper data

To manage these volumes:

```bash
# List all volumes
docker volume ls | grep local_

# Remove a specific volume (e.g., to reset data)
docker volume rm local_postgres-data

# Remove all project volumes (WARNING: This deletes all data)
docker compose down -v
```

### Kafka Container Failing

When migrating from the old local setup, the Kafka container might fail with a
`InconsistentClusterIdException`. Fix this by running
`docker volume rm local_kafka-data`.

## Development

For development purposes, you can modify the `docker-compose.yaml` file to:

- Change port mappings
- Add environment variables
- Mount additional volumes
- Add new services

---
> Source: [gentrace/gentrace](https://github.com/gentrace/gentrace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
