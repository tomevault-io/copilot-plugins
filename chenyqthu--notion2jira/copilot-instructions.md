## notion2jira

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Notion2JIRA is a bi-directional synchronization system between Notion Database and JIRA projects, designed for EBG commercial product teams. It consists of two main services:

1. **webhook-server**: Public-facing Node.js service that receives Notion webhooks and queues sync tasks
2. **sync-service**: Internal Python service that processes sync tasks and communicates with JIRA

## Architecture

The system uses a decoupled architecture with Redis as message queue:
```
Notion → webhook-server (Node.js) → Redis Queue → sync-service (Python) → JIRA
```

## Development Commands

### Webhook Server (Node.js)
```bash
cd webhook-server
npm install                    # Install dependencies
npm start                      # Start production server
npm run dev                    # Start with nodemon (development)
npm test                       # Run Jest tests
```

### Sync Service (Python)
```bash
cd sync-service
pip install -r requirements.txt    # Install Python dependencies
python main.py                     # Start sync service
./start.sh                         # Start with environment setup
./start.sh --check                 # Environment check only
./start.sh --setup-only            # Setup environment only
```

### Full System Deployment
```bash
./deploy.sh                        # Complete system deployment
./deploy.sh --check-only           # Environment check only
./deploy.sh --start-only           # Start services only
```

## Environment Setup

1. Copy `.env_example` to `.env` and configure required variables:
   - `JIRA_BASE_URL`, `JIRA_USERNAME`, `JIRA_PASSWORD`
   - `NOTION_TOKEN`, `NOTION_DATABASE_ID`
   - `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD` (optional)

2. Start Redis server (required for both services)

3. Services can be started independently but both need Redis

## Key Components

### Webhook Server (`webhook-server/`)
- **server.js**: Main Express application with security, CORS, and rate limiting
- **routes/webhook.js**: Notion webhook endpoint handler
- **routes/admin.js**: Admin management endpoints
- **config/redis.js**: Redis client configuration
- **ecosystem.config.js**: PM2 process configuration

### Sync Service (`sync-service/`)
- **main.py**: Main application entry point with service orchestration
- **services/sync_service.py**: Core synchronization logic
- **services/field_mapper.py**: Notion-JIRA field mapping and transformation
- **services/jira_client.py**: JIRA REST API client
- **services/notion_client.py**: Notion API client
- **services/redis_client.py**: Async Redis client for queue management
- **services/version_mapper.py**: Project version mapping logic
- **config/settings.py**: Configuration management
- **config/logger.py**: Logging setup

## Sync Triggers

The system supports multiple sync trigger mechanisms:
1. **Button Property** (recommended): User clicks button in Notion page
2. **Checkbox Control**: Fields named `sync2jira`, `同步到JIRA`, `Sync to JIRA`
3. **Default Strategy**: Any webhook triggers sync

## Field Mapping

Key field mappings from Notion to JIRA:
- `title` → `summary`
- `description` + `ai_summary` → `description` (combined)
- Priority mapping: `高 High` → Priority ID 1, etc.
- Status mapping: `DEVING` → `开发中`, `已发布 DONE` → `完成`
- Version extraction from Formula fields or Relation API calls

## Remote Link Management

Uses GlobalId mechanism to prevent duplicate links:
- `notion-page-{hash}`: Original requirement page link
- `notion-prd-{hash}`: PRD document link
- Links are updated rather than recreated for same URLs

## Testing

- **Webhook Server**: Uses Jest for unit tests (`npm test`)
- **Sync Service**: Basic tests in Python files
- **Integration**: End-to-end sync flow testing available

## Monitoring and Debugging

### Health Checks
```bash
curl http://localhost:7654/health     # Webhook server health
curl http://localhost:7654/stats      # System statistics
```

### Logs
```bash
tail -f webhook-server/logs/webhook.log    # Webhook service logs
tail -f sync-service/logs/sync_service.log # Sync service logs
```

### Redis Queue Management
```bash
redis-cli
LLEN notion_sync_queue              # Check queue length
LRANGE notion_sync_queue 0 -1       # View queue contents
```

## Performance Optimizations

- **Formula Field Version Extraction**: 6800x performance improvement (0.0001s vs 0.6844s)
- **Dual Storage Strategy**: Raw + parsed properties for data integrity
- **Async Processing**: Non-blocking sync operations
- **Redis Caching**: User mappings and metadata caching

## Security

- **CORS Configuration**: Configurable allowed origins
- **Rate Limiting**: Configurable per-IP request limits
- **Helmet.js**: Security headers and CSP
- **Input Validation**: Webhook data validation
- **Environment Variables**: Sensitive data in `.env` files

## Deployment

The system supports multiple deployment approaches:
1. **Traditional**: Separate Node.js and Python services
2. **Docker**: Container-based deployment with docker-compose
3. **PM2**: Process management for production environments

Services communicate via Redis message queues, allowing independent scaling and deployment.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ChenyqThu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
