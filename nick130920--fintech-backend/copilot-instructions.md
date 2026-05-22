## backend-architecture

> Backend architecture and clean code principles for Go fintech application


# Backend Architecture Rules

## Clean Architecture Structure
- Follow the established directory structure: `internal/entity`, `internal/usecase`, `internal/controller/http/v1`, `pkg/repository`
- Entities in [internal/entity](mdc:internal/entity) define domain models
- Use cases in [internal/usecase](mdc:internal/usecase) contain business logic
- Controllers in [internal/controller/http/v1](mdc:internal/controller/http/v1) handle HTTP requests
- Repositories in [pkg/repository](mdc:pkg/repository) implement data access

## Error Handling
- Use structured errors from [pkg/apperrors/errors.go](mdc:pkg/apperrors/errors.go)
- Always use `AppError` struct instead of generic `errors.New()`
- Include proper error codes and context with `WithDetails()` and `WithInternal()`

## API Standards
- DTOs must be in [internal/controller/http/v1/dto](mdc:internal/controller/http/v1/dto)
- Use proper HTTP status codes: 200 (OK), 201 (Created), 400 (Bad Request), 401 (Unauthorized), 404 (Not Found), 500 (Internal Error)
- Always validate request data using the validator from [pkg/validator](mdc:pkg/validator)

## Database Operations
- Use GORM for database operations
- Always use transactions for complex operations
- Preload related entities to avoid N+1 queries
- Follow the repository pattern defined in [internal/usecase/repo](mdc:internal/usecase/repo)

## Security
- JWT authentication is handled in [pkg/auth/jwt.go](mdc:pkg/auth/jwt.go)
- Always validate user permissions before operations
- Extract user ID from authenticated context: `userID, exists := c.Get("user_id")`
- Never expose sensitive information in logs or responses

---
> Source: [nick130920/fintech-backend](https://github.com/nick130920/fintech-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
