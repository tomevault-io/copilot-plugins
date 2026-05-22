## deployment-railway

> Railway deployment configuration and environment setup


# Railway Deployment Rules

## Configuration Files
- [railway.toml](mdc:railway.toml) configures Railway deployment
- Set `GIN_MODE=release` for production logging
- Use Dockerfile builder for containerized deployment

## Environment Variables
- `GIN_MODE=release` enables production optimizations (definir en Railway Dashboard si los logs muestran "debug")
- [configs/config.go](mdc:configs/config.go) usa por defecto `release` si no está definido
- [railway.toml](mdc:railway.toml) define `GIN_MODE=release` en `[env]` para el servicio
- Database connection via Railway's PostgreSQL service
- JWT secrets and other sensitive data via Railway environment

## Long-running endpoints
- `analyze-sms-batch`: **100 SMS**, **4 chunks**; contexto servidor y cliente **~10 min** (429 + lentitud del modelo gratuito)
- Modelo gratuito OpenRouter puede 429; hay reintentos con backoff en código

## Logging Configuration
- Production mode uses JSON logging format
- Compact request/response logging for Railway console
- Structured error reporting with request IDs

## Database Migrations
- GORM AutoMigrate handles schema changes automatically
- No manual migration scripts needed for Railway
- Database initialization in [database.go](mdc:pkg/database/database.go)

## Build Process
```dockerfile
RUN CGO_ENABLED=0 GOOS=linux go build -tags netgo -ldflags '-s -w' -o app ./cmd/server
```

## Health Checks
- Implement health check endpoints for Railway monitoring
- Database connectivity verification
- Service status reporting

## Security Headers
- CORS configuration for frontend integration
- Security headers middleware for production
- Rate limiting for API protection

---
> Source: [nick130920/fintech-backend](https://github.com/nick130920/fintech-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
