## logging-middleware

> Logging and middleware configuration for different environments


# Logging and Middleware Rules

## Environment-Based Configuration
- Development (`GIN_MODE=debug`): Use [EnhancedLoggerMiddleware](mdc:internal/controller/http/v1/middleware_logger.go) for detailed logging
- Production (`GIN_MODE=release`): Use [RailwayLoggerMiddleware](mdc:internal/controller/http/v1/middleware_logger_railway.go) for compact logging

## Logging Standards
- Configure logrus in [setupLogging()](mdc:internal/app/app.go) function
- Production logs use JSON format for Railway compatibility
- Development logs use colored text format
- Always include request IDs for traceability
- Sanitize sensitive fields (password, token, secret, key, authorization)

## Middleware Stack Order
1. Recovery middleware (environment-specific)
2. Security headers
3. CORS
4. Logging middleware (environment-specific)
5. Error handling
6. Rate limiting
7. Validation
8. Authentication (for protected routes)

## Request ID Generation
- Use short UUIDs (8 characters) for compact logs
- Set in context: `c.Set("request_id", requestID)`
- Include in all log messages: `[requestID]`

## Error Logging
- Use structured logging with logrus
- Include request context (method, path, IP, user)
- Log slow requests (>2s) with special warning
- Log large responses (>1MB) with special warning

---
> Source: [nick130920/fintech-backend](https://github.com/nick130920/fintech-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
