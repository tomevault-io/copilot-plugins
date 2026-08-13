## postgresql-rules

> You are working with PostgreSQL database and Drizzle ORM (latest stable version). Follow these rules for type-safe, performant, and maintainable database operations.


# PostgreSQL & Drizzle ORM Development Rules

## Project Context

You are working with PostgreSQL database and Drizzle ORM (latest stable version). Follow these rules for type-safe, performant, and maintainable database operations.

## Drizzle ORM Latest Stable Guidelines

### Core Principles

- Always use the latest stable Drizzle ORM syntax and patterns
- Leverage TypeScript for full type safety
- Use Drizzle's query builder for complex operations
- Prefer Drizzle's native methods over raw SQL when possible
- Utilize Drizzle's migration system for schema changes

#### Column Types & Constraints

- Use appropriate PostgreSQL column types:
  - `uuid()` for primary keys and foreign keys
  - `varchar(length)` for strings with known max length
  - `text()` for unlimited text
  - `timestamp({ withTimezone: true })` for dates
  - `jsonb()` for JSON data
  - `decimal(precision, scale)` for precise numbers
  - `boolean()` for true/false values
  - `serial()` or `bigserial()` only when auto-incrementing is needed

#### Migration Commands

```bash
# Generate migrations
npx drizzle-kit generate:pg

# Apply migrations
npx drizzle-kit push:pg

# Check migration status
npx drizzle-kit up:pg
```

### Error Handling & Validation

#### Database Errors

```typescript
// Handle common database errors
const handleDatabaseError = (error: any) => {
  if (error.code === '23505') {
    // Unique violation
    throw new Error('Resource already exists');
  }
  if (error.code === '23503') {
    // Foreign key violation
    throw new Error('Referenced resource does not exist');
  }
  if (error.code === '23514') {
    // Check violation
    throw new Error('Data violates constraints');
  }
  throw new Error(`Database error: ${error.message}`);
};
```

#### Input Validation

```typescript
// Use Zod for validation with Drizzle schemas
import { z } from 'zod';

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1).max(100),
});

const validateAndCreateUser = async (input: unknown) => {
  const validatedData = createUserSchema.parse(input);
  return await createUser(validatedData);
};
```

### Performance Optimization

#### Query Optimization

- Use `select()` with specific fields instead of `select()`
- Implement proper indexing for frequently queried columns
- Use `limit()` and `offset()` for pagination
- Leverage `exists()` for existence checks instead of counting
- Use `batch()` for multiple inserts/updates

#### Connection Management

- Use connection pooling in production
- Close connections properly in serverless environments
- Monitor connection usage and adjust pool size accordingly
- Use read replicas for read-heavy operations when available

### Security Best Practices

#### SQL Injection Prevention

- Always use parameterized queries (Drizzle handles this automatically)
- Never concatenate user input into SQL strings
- Use Drizzle's query builder methods instead of raw SQL when possible
- Validate and sanitize all user inputs

### Development Workflow

#### Schema Changes

1. Update schema files
2. Generate migration: `npx drizzle-kit generate:pg`
3. Review generated SQL
4. Apply migration: `npx drizzle-kit push:pg`
5. Update TypeScript types if needed

#### Code Review Checklist

- [ ] Proper error handling implemented
- [ ] Indexes added for query performance
- [ ] Foreign key constraints defined
- [ ] Input validation in place
- [ ] TypeScript types properly inferred
- [ ] Transactions used where appropriate
- [ ] Connection pooling configured
- [ ] Migration files reviewed

Remember: Always prioritize type safety, performance, and maintainability. Use Drizzle's TypeScript integration to catch errors at compile time rather than runtime.

---
> Source: [ronnypdev/Notes-Web](https://github.com/ronnypdev/Notes-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
