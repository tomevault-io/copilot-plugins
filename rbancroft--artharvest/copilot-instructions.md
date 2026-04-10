## artharvest

> This document provides context for Claude Code sessions to maintain continuity and understanding of the ArthArvest project.

# Claude Context - ArthArvest Project

This document provides context for Claude Code sessions to maintain continuity and understanding of the ArthArvest project.

## Project Overview

**ArthArvest** is a Go application that monitors X/Twitter lists for art content and automatically archives images with metadata. The application follows 12-factor app principles and is designed for containerized deployment.

## Key Design Decisions

### Architecture Patterns
- **12-Factor App**: Environment-based configuration, stateless processes
- **Clean Architecture**: Separated concerns with internal packages
- **Service Layer Pattern**: Business logic isolated in services
- **Repository Pattern**: Data access abstracted through database layer

### Technology Stack
- **Language**: Go 1.21
- **Database**: PostgreSQL 15 with JSONB for extensible metadata
- **Web Framework**: Gin for REST API
- **HTTP Client**: Standard library with rate limiting via golang.org/x/time/rate
- **Logging**: Structured JSON logging with logrus
- **Containerization**: Docker with multi-stage builds

### API Strategy
- **Twitter API v2**: Free tier compatible (300 req/15min, 1,500 tweets/month)
- **List-based Monitoring**: More efficient than account-by-account monitoring
- **Rate Limiting**: Respects API constraints with token bucket algorithm

## Project Structure

```
artharvest-app/
├── cmd/artharvest/main.go          # Application entry point
├── internal/
│   ├── app/app.go                  # Application bootstrap & lifecycle
│   ├── api/                        # REST API layer
│   │   ├── routes.go               # Route definitions
│   │   └── handlers.go             # HTTP handlers
│   ├── config/config.go            # Configuration management
│   ├── database/                   # Data layer
│   │   ├── database.go             # Connection pooling
│   │   └── schema.sql              # Database schema
│   ├── logger/logger.go            # Structured logging
│   ├── models/models.go            # Data models
│   ├── scheduler/scheduler.go      # Periodic job execution
│   └── services/                   # Business logic
│       ├── services.go             # Service container
│       ├── twitter.go              # X/Twitter API client
│       ├── media.go                # Media download & storage
│       ├── lists.go                # List management
│       └── processing.go           # Data processing
├── docker-compose.yml              # Container orchestration
├── Dockerfile                      # Application container
├── .env.example                    # Environment variable template
└── config.example.yaml             # YAML configuration template
```

## Key Components

### Configuration System (`internal/config/`)
- Environment-first with YAML override capability
- Validation for required fields
- Type-safe configuration with structs
- Duration parsing for intervals and timeouts

### Database Schema (`internal/database/schema.sql`)
- **twitter_lists**: Monitored lists with activation control
- **tweets**: Tweet metadata with JSONB for extensibility
- **media_items**: Downloaded media with file paths and SHA256 hashes
- **processing_jobs**: Background job tracking and status

### Services Layer (`internal/services/`)
- **TwitterService**: API client with rate limiting
- **MediaService**: Download management with resume capability
- **ListService**: CRUD operations for Twitter lists
- **ProcessingService**: Data storage and retrieval

### Scheduler (`internal/scheduler/`)
- Configurable interval execution
- Graceful shutdown handling
- Error recovery and logging
- List processing with rate limiting between requests

## Current Implementation Status

### Completed Features ✅
- [x] Go module and project structure
- [x] Configuration system with env/YAML support
- [x] PostgreSQL database schema and connection pooling
- [x] Twitter API v2 client with rate limiting
- [x] Media download service with resume capability
- [x] Duplicate detection via SHA256 hashing
- [x] REST API skeleton with list management
- [x] Scheduled execution daemon
- [x] Structured JSON logging
- [x] Docker containerization with compose

### Placeholder Implementations 🚧
- [ ] Complete API handlers for tweets and media endpoints
- [ ] Processing job management and status tracking
- [ ] Media classification system (future enhancement)
- [ ] Web UI for list and media management
- [ ] Comprehensive test suite

### Known Technical Debt
- Some API handlers return "Not implemented yet"
- Missing integration tests
- No migration system for database schema changes
- Rate limiting needs monitoring and alerting

## Development Guidelines

### Code Conventions
- Follow standard Go conventions and gofmt
- Use structured logging with key-value pairs
- Implement graceful shutdowns for all services
- Handle contexts for cancellation and timeouts

### Database Patterns
- Use parameterized queries to prevent SQL injection
- Leverage JSONB for extensible metadata storage
- Create indexes for frequently queried columns
- Use transactions for multi-table operations

### Error Handling
- Wrap errors with context using fmt.Errorf
- Log errors at appropriate levels
- Return structured API errors
- Implement retry logic for transient failures

### Testing Strategy
- Unit tests for service layer logic
- Integration tests for database operations
- API tests for HTTP endpoints
- Mock external dependencies (Twitter API)

## Environment Variables Reference

### Required
```
TWITTER_BEARER_TOKEN     # X/Twitter API bearer token
POSTGRES_PASSWORD        # Database password
```

### Optional with Defaults
```
ARTHARVEST_HOST=0.0.0.0
ARTHARVEST_PORT=8080
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=artharvest
POSTGRES_USER=artharvest
POSTGRES_SSLMODE=prefer
LOG_LEVEL=info
SCHEDULER_ENABLED=true
SCHEDULER_INTERVAL=24h
TWITTER_LOOKBACK_WINDOW=6h
TWITTER_RATE_LIMIT_REQUESTS=300
TWITTER_RATE_LIMIT_WINDOW=15m
STORAGE_BASE_PATH=./data
```

## Common Development Tasks

### Adding New API Endpoints
1. Define route in `internal/api/routes.go`
2. Implement handler in `internal/api/handlers.go`
3. Add service method if needed
4. Update database schema if required

### Extending Database Schema
1. Update `internal/database/schema.sql`
2. Add fields to models in `internal/models/models.go`
3. Update service methods for new fields
4. Consider backward compatibility

### Adding New Services
1. Create service file in `internal/services/`
2. Define interface and implementation
3. Add to service container in `internal/services/services.go`
4. Wire into application in `internal/app/app.go`

## Troubleshooting

### Common Issues
- **Rate limiting**: Check Twitter API usage in logs
- **Database connection**: Verify PostgreSQL is running and credentials are correct
- **File permissions**: Ensure data directory is writable
- **Memory usage**: Monitor for large file downloads

### Debug Configuration
```bash
LOG_LEVEL=debug
SCHEDULER_INTERVAL=5m  # More frequent for testing
TWITTER_LOOKBACK_WINDOW=1h  # Shorter window for testing
```

## Future Enhancements

### Planned Features
- Image classification using machine learning
- Web dashboard for browsing archived content
- Webhook notifications for new content
- Content search and tagging system
- Export capabilities (ZIP, RSS, etc.)

### Technical Improvements
- Implement comprehensive metrics collection
- Add distributed tracing for debugging
- Create database migration system
- Build comprehensive test suite
- Add performance monitoring and alerting

This context document should be updated as the project evolves to maintain accuracy for future Claude Code sessions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rbancroft)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rbancroft)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
