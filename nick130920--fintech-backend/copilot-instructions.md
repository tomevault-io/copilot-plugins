## api-routes

> API routing and handler patterns for fintech backend


# API Routes and Handler Rules

## Route Registration
- Register routes in [NewRouter()](mdc:internal/controller/http/v1/router.go)
- Define both routes with and without trailing slashes to prevent 307 redirects
- Group related routes (e.g., `/api/v1/bank-accounts`, `/api/v1/notification-patterns`)

## Handler Patterns
- Extract user ID from context: `userID, exists := c.Get("user_id")`
- Validate request DTOs before processing
- Use structured error responses with request IDs
- Return appropriate HTTP status codes

## Bank Account Routes
```go
// Both with and without trailing slash
bankAccountsGroup.GET("/", handler.GetUserBankAccounts)
bankAccountsGroup.GET("", handler.GetUserBankAccounts)
bankAccountsGroup.POST("/", handler.CreateBankAccount)
bankAccountsGroup.POST("", handler.CreateBankAccount)
```

## Authentication Middleware
- Apply `AuthMiddleware()` to protected route groups
- Public routes (health, swagger) don't need authentication
- Extract and validate JWT tokens

## Error Responses
```go
c.JSON(status, gin.H{
    "error": errorCode,
    "message": errorMessage,
    "request_id": requestID,
})
```

## Validation
- Bind JSON request: `c.ShouldBindJSON(&req)`
- Validate using custom validator
- Return 400 for validation errors
- Include field-specific error messages

---
> Source: [nick130920/fintech-backend](https://github.com/nick130920/fintech-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
