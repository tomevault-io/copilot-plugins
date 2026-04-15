## algolia-sync

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

algolia-sync is a Go service that syncs anime search data from message queues (Pulsar/Kafka) to Algolia search index. It processes anime records with create, update, and delete operations.

## Architecture

The application follows a layered architecture with Redis as an intermediate storage:

- **CMD Layer**: Entry point in `cmd/main.go` using Cobra CLI framework
- **Commands**: CLI commands in `internal/commands/` for different modes:
  - `serve-algolia-sync`: Pulsar-based message consumption (stores in Redis)
  - `serve-algolia-sync-kafka`: Kafka-based message consumption (stores in Redis)
  - `sync-redis-to-algolia`: Cron job command that syncs Redis data to Algolia
- **Eventing Layer**: Message queue handlers in `internal/eventing/`
- **Services Layer**: Business logic in `internal/services/`:
  - `redis/`: Redis client wrapper for intermediate storage
  - `redis_processor/`: Core processing logic for anime records (Redis storage)
  - `redis_processor_kafka/`: Kafka-specific processing (Redis storage)
  - `algolia/`: Algolia client wrapper (used by cron job)
  - `processor/`: Generic message processing utilities
- **Config**: Configuration management in `config/` using jinzhu/configor

## Development Commands

### Building
```bash
go build ./cmd/
```

### Testing
```bash
go test ./...
```

### Running the Service
```bash
# Pulsar-based sync (stores data in Redis)
go run ./cmd serve-algolia-sync

# Kafka-based sync (stores data in Redis)
go run ./cmd serve-algolia-sync-kafka

# Cron job to sync Redis data to Algolia (run this as a scheduled job)
go run ./cmd sync-redis-to-algolia
```

### Generate Mocks
```bash
make mocks
```

### Database Migrations
```bash
make migrate-create name=migration_name
```

## Configuration

The application uses a configuration file at `config/config.dev.json` and environment variables:

- **Pulsar**: `PULSARURL`, `PULSARTOPIC`, `PULSARSUBSCRIPTIONNAME`
- **Algolia**: `ALGOLIA_APP_ID`, `ALGOLIA_API_KEY`, `ALGOLIA_INDEX`, `ALGOLIA_FLUSH_TIMEOUT`
- **Kafka**: `KAFKA_CONSUMER_GROUP_NAME`, `KAFKA_BOOTSTRAP_SERVERS`, `KAFKA_TOPIC`, `KAFKA_OFFSET`
- **Redis**: `REDIS_URL`, `REDIS_PASSWORD`, `REDIS_DB`, `REDIS_KEY`

## Key Dependencies

- **Message Queues**: Apache Pulsar client and Confluent Kafka client
- **Search**: Algolia Go client
- **Storage**: Redis Go client
- **CLI**: Cobra framework
- **Logging**: Uber Zap
- **Config**: jinzhu/configor

## Data Processing Flow

### Real-time Message Processing
1. Messages arrive via Pulsar or Kafka containing anime record changes
2. `eventing` layer receives and deserializes messages into `Payload` structs
3. `redis_processor` determines action type (create/update/delete) and transforms data
4. Processed data is stored in Redis as `QueuedItem` objects with timestamps
5. Processing includes URL encoding for anime titles and ObjectID management

### Batch Sync to Algolia (Cron Job)
1. `sync-redis-to-algolia` command reads all queued items from Redis
2. Items are batch-processed and sent to Algolia search index
3. On successful sync, Redis queue is cleared
4. Failed items remain in queue for retry on next cron run

This architecture decouples message processing from Algolia sync, providing better reliability and allowing for batch optimizations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/weeb-vip)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/weeb-vip)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
