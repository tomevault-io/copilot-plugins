## rust-flextide-automations

> Defines database migration standards and best practices using sqlx-cli for MySQL and PostgreSQL support. Ensures consistent migration management across the codebase.


# Database Migrations Rules (sqlx-cli)

## Overview

This project uses [sqlx-cli](https://github.com/launchbadge/sqlx/tree/main/sqlx-cli) for database migrations, supporting both **MySQL** and **PostgreSQL** databases.

## Migration Location

All database migrations are stored in the `/backend/migrations` directory.

## Setup and Configuration

### Environment Variables

Database connection is configured via `DATABASE_URL` environment variable:
- **PostgreSQL**: `postgres://username:password@localhost/database_name`
- **MySQL**: `mysql://username:password@localhost/database_name`

Store `DATABASE_URL` in `backend/.env` (gitignored) and provide `backend/.env.sample` as a template.

### sqlx-cli Installation

sqlx-cli should be installed with support for both databases:
```bash
cargo install sqlx-cli --no-default-features --features native-tls,postgres,mysql
```

## Migration File Naming

Migrations follow the pattern: `{timestamp}_{description}.sql`

Example: `20250115120000_create_users_table.sql`

## Creating Migrations

### Add a new migration:
```bash
# From backend/ directory
cd backend
sqlx migrate add <migration_name>

# Or from project root
sqlx migrate add <migration_name> --source backend/migrations
```

This creates a new migration file in `backend/migrations/` with a timestamp prefix.

### Migration File Structure

Each migration file should:
1. Include comments describing the purpose
2. **Use database-agnostic SQL that works on both MySQL and PostgreSQL**
3. **Never include commented-out database-specific code** - write SQL that works for both databases
4. Be idempotent (use `IF NOT EXISTS` where appropriate)
5. Handle `updated_at` timestamps in application code rather than using database-specific features like `ON UPDATE CURRENT_TIMESTAMP` (MySQL) or triggers (PostgreSQL)

Example:
```sql
-- Create users table
-- Supports both MySQL and PostgreSQL

-- PostgreSQL version
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

-- MySQL version (if different)
-- CREATE TABLE IF NOT EXISTS users (
--     id INT AUTO_INCREMENT PRIMARY KEY,
--     email VARCHAR(255) NOT NULL UNIQUE,
--     created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
-- );
```

## Running Migrations

### Apply all pending migrations:
```bash
# From backend/ directory
cd backend
sqlx migrate run

# Or from project root
sqlx migrate run --source backend/migrations
```

### Revert the last migration:
```bash
# From backend/ directory
cd backend
sqlx migrate revert

# Or from project root
sqlx migrate revert --source backend/migrations
```

### Check migration status:
```bash
# From backend/ directory
cd backend
sqlx migrate info

# Or from project root
sqlx migrate info --source backend/migrations
```

## Offline Mode / Compile-Time Query Verification

sqlx can verify SQL queries at compile time without requiring a database connection. This is useful for CI/CD and offline development.

### Prepare query metadata:
```bash
# From backend/ directory (for workspace)
cd backend
cargo sqlx prepare --workspace -- --bin api

# Or for a specific crate
cd backend/crates/api
cargo sqlx prepare
```

This generates query metadata in `.sqlx/` directory that allows compilation without `DATABASE_URL`.

### Check if metadata is up-to-date:
```bash
cargo sqlx prepare --check
```

**Note**: The `prepare` subcommand must be invoked as `cargo sqlx prepare`, not `sqlx prepare`.

## Programmatic Migration Execution

In Rust code, run migrations at application startup:

```rust
use sqlx::{Pool, Postgres, MySql};

// For PostgreSQL
let pool = Pool::<Postgres>::connect(&std::env::var("DATABASE_URL")?).await?;
sqlx::migrate!("./migrations").run(&pool).await?;

// For MySQL
// let pool = Pool::<MySql>::connect(&std::env::var("DATABASE_URL")?).await?;
// sqlx::migrate!("./migrations").run(&pool).await?;
```

**Note**: When running from the `backend/` directory, use `"./migrations"` as the path. If running from project root, use `"./backend/migrations"`.

## Module Table Naming Convention

All database tables that belong to a module must use the prefix `module_<module_name>_`.

**Examples:**
- CRM module tables: `module_crm_customers`, `module_crm_customer_notes`
- Other module tables: `module_<module_name>_<table_name>`

This convention ensures:
- Clear module ownership and organization
- Avoids naming conflicts between modules
- Makes it easy to identify module-specific tables
- Supports multi-module architecture

**Note**: Core platform tables (like `users`, `organizations`, `organization_members`) do not use this prefix as they are part of the core system, not a module.

## Best Practices

1. **Always test migrations** on both MySQL and PostgreSQL if supporting both
2. **Never modify existing migrations** - create new migrations to fix issues
3. **Use transactions** for complex migrations when possible
4. **Document breaking changes** in migration comments
5. **Keep migrations small and focused** - one logical change per migration
6. **Use descriptive names** that explain what the migration does
7. **Test migrations** in development before applying to production
8. **Back up databases** before running migrations in production
9. **Use module table prefixes** for all module-specific tables (`module_<module_name>_<table_name>`)

## Database-Specific Considerations

### MySQL
- Use `AUTO_INCREMENT` for primary keys
- Use `VARCHAR(255)` for text fields
- Use `TIMESTAMP` or `DATETIME` for dates
- Case-sensitive table names depend on OS/filesystem

### PostgreSQL
- Use `SERIAL` or `BIGSERIAL` for auto-incrementing IDs
- Use `TEXT` or `VARCHAR(n)` for text fields
- Use `TIMESTAMP` or `TIMESTAMPTZ` for dates
- Table names are case-insensitive (converted to lowercase)

### Cross-Database Compatibility

When writing migrations that work for both:
- Use standard SQL types when possible
- Avoid database-specific functions
- Test thoroughly on both databases
- Consider using conditional SQL if necessary

## Migration Workflow

1. **Create migration** (from `backend/` directory):
   ```bash
   cd backend
   sqlx migrate add <name>
   ```
   Or from project root: `sqlx migrate add <name> --source backend/migrations`

2. **Write SQL**: Edit the generated migration file in `backend/migrations/`

3. **Test locally** (from `backend/` directory):
   ```bash
   cd backend
   sqlx migrate run
   ```
   Or from project root: `sqlx migrate run --source backend/migrations`

4. **Commit**: Add migration file to version control

5. **Deploy**: Migrations run automatically at application startup or manually via CI/CD

## References

- [sqlx-cli on crates.io](https://crates.io/crates/sqlx-cli)
- [sqlx-cli GitHub](https://github.com/launchbadge/sqlx/tree/main/sqlx-cli)
- [sqlx Documentation](https://docs.rs/sqlx/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JuKu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
