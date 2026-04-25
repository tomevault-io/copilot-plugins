## nextboiler

> Database queries with prisma in repository files

# Database Operations with Prisma ORM

This document outlines the standards and best practices for implementing database operations using Prisma ORM in this project.

## Core Principles

1.  **Utilize Prisma Client:** All database interactions should be handled through the generated Prisma Client. Avoid raw SQL (`$queryRaw`, `$executeRaw`) unless absolutely necessary for performance-critical or highly complex scenarios not easily expressible with Prisma Client.
2.  **Prioritize Type Safety:** Leverage Prisma's generated types based on your `packages/db/prisma/schema.prisma` file to ensure end-to-end type safety from your database to your application code.
3.  **Schema as Single Source of Truth:** The `packages/db/prisma/schema.prisma` file is the definitive source for your database schema. Use Prisma Migrate for all schema changes and database migrations.
4.  **Maintainability & Readability:** Write clean, understandable, and maintainable query code. Use meaningful variable names and structure queries logically.

## Query Implementation Guidelines

### Basic CRUD Operations

Use the generated Prisma Client methods for standard Create, Read, Update, and Delete operations.

```typescript
// Example: Finding a user by ID
import { db } from "@package/db"

async function getUserById(userId: number) {
  const user = await db.user.findUnique({
    where: { id: userId },
  });
  return user;
}

// Example: Creating a user
async function createUser(name: string, email: string) {
  const newUser = await db.user.create({
    data: { name, email },
  });
  return newUser;
}

// Example: Updating a user
async function updateUserEmail(db: number, newEmail: string) {
  const updatedUser = await prisma.user.update({
    where: { id: userId },
    data: { email: newEmail },
  });
  return updatedUser;
}

// Example: Deleting a user
async function deleteUser(userId: number) {
  const deletedUser = await db.user.delete({
    where: { id: userId },
  });
  return deletedUser;
}
```

### Relations

Utilize Prisma's relation query options (e.g., `include`, nested writes) to work with related data efficiently. Define relations clearly in your `packages/db/prisma/schema.prisma`.

```typescript
// Example: Fetching a user and their posts
async function getUserWithPosts(userId: number) {
  const userWithPosts = await db.user.findUnique({
    where: { id: userId },
    include: { posts: true }, // Assumes a relation 'posts' is defined in schema.prisma
  });
  return userWithPosts;
}
```

### Type Safety

Ensure all function inputs and outputs interacting with Prisma Client are correctly typed using the automatically generated types from Prisma. Avoid using `any` for data coming from or going to the database.

### Raw SQL

Reserve raw SQL functions (`$queryRaw`, `$executeRaw`, `$queryRawUnsafe`, `$executeRawUnsafe`) for situations where Prisma Client's API is insufficient or significantly less performant. Always prefer parameterized raw queries (`$queryRaw`, `$executeRaw`) over unsafe variants to prevent SQL injection vulnerabilities.

## Schema Management

-   Define your models, relations, and enums in `packages/db/prisma/schema.prisma`.
-   Use `pnpm db:migrate` during development to evolve your schema and apply migrations.
-   Generate Prisma Client using `pnpm db:generate` after any schema changes.

## Anti-patterns to Avoid

1.  ❌ Writing raw SQL for operations easily achievable with Prisma Client.
2.  ❌ Bypassing Prisma's generated types (e.g., using `any` excessively).
3.  ❌ Manually altering the database schema without using Prisma Migrate.
4.  ❌ Not regenerating Prisma Client after schema changes (`pnpm db:generate`).
5.  ❌ Writing overly complex single queries where multiple, simpler queries might be more readable and maintainable, unless performance dictates otherwise.

**Goal:** Write type-safe, efficient, and maintainable database access code using Prisma ORM and Prisma Client.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/P156HAM) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
