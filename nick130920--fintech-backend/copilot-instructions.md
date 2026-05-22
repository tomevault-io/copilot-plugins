## database-entities

> Database entity and repository patterns for GORM


# Database Entity and Repository Rules

## Entity Definitions
- Define entities in [internal/entity](mdc:internal/entity) directory
- Use GORM tags for database mapping
- Include `CreatedAt`, `UpdatedAt`, `DeletedAt` for soft deletes
- Use proper foreign key relationships

## Repository Pattern
- Interface definitions in [internal/usecase/repo](mdc:internal/usecase/repo)
- Implementations in [pkg/repository](mdc:pkg/repository)
- Use GORM for database operations
- Handle errors properly with structured `AppError`

## Bank Account Entity
- Reference [BankAccount](mdc:internal/entity/bank_account.go) for structure
- Include user association: `UserID uint`
- Support notification settings and balance tracking

## Transaction Entity
- Reference [Transaction](mdc:internal/entity/transaction.go) for extended fields
- Include bank account association: `BankAccountID *uint`
- Support validation status and AI confidence

## Migration Handling
- Use GORM AutoMigrate in [database.go](mdc:pkg/database/database.go)
- Add new entities to `runMigrations()` function
- Include in `DropTables()` for cleanup (reverse dependency order)

## Query Patterns
```go
// Preload relationships
db.Preload("Category").Where("user_id = ?", userID).Find(&expenses)

// Use transactions for complex operations
tx := r.db.Begin()
defer func() {
    if r := recover(); r != nil {
        tx.Rollback()
    }
}()
```

---
> Source: [nick130920/fintech-backend](https://github.com/nick130920/fintech-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
