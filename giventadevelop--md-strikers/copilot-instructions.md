## prisma

> - **Prisma Client Initialization**

# Prisma Usage Rules

- **Prisma Client Initialization**
  - ✅ DO: Use the standard singleton pattern in `src/lib/prisma.ts`
    ```typescript
    import { PrismaClient } from '@prisma/client'

    const globalForPrisma = globalThis as unknown as {
      prisma: PrismaClient | undefined
    }

    export const prisma =
      globalForPrisma.prisma ??
      new PrismaClient({
        log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
      })

    if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
    ```
  - ❌ DON'T: Create multiple Prisma client instances
  - ❌ DON'T: Initialize Prisma client without proper global caching
  - ❌ DON'T: Use direct instantiation in route handlers or components

- **Runtime Configuration**
  - ✅ DO: Always use Node.js runtime for routes with Prisma
    ```typescript
    // Required for Prisma compatibility
    export const runtime = 'nodejs'
    ```
  - ❌ DON'T: Use Edge runtime with Prisma operations
  - ❌ DON'T: Omit runtime configuration in API routes using Prisma

- **Import Requirements**
  - ✅ DO: Import Prisma client from the centralized location
    ```typescript
    import { prisma } from '@/lib/prisma'
    ```
  - ❌ DON'T: Import PrismaClient directly in route files
  - ❌ DON'T: Create new PrismaClient instances outside `src/lib/prisma.ts`

- **Error Handling**
  - ✅ DO: Use try-catch blocks around Prisma operations
    ```typescript
    try {
      const result = await prisma.user.findMany()
    } catch (error) {
      console.error('Database error:', error)
      throw new Error('Failed to fetch users')
    }
    ```
  - ✅ DO: Log database errors with proper context
  - ❌ DON'T: Expose raw Prisma errors to clients

- **Transaction Handling**
  - ✅ DO: Use transactions for multiple operations
    ```typescript
    await prisma.$transaction([
      prisma.user.create({ data: userData }),
      prisma.log.create({ data: logData })
    ])
    ```
  - ❌ DON'T: Perform multiple dependent operations without transactions

- **Performance Considerations**
  - ✅ DO: Use proper select and include statements
    ```typescript
    const user = await prisma.user.findUnique({
      where: { id },
      select: { id: true, email: true }
    })
    ```
  - ❌ DON'T: Fetch unnecessary fields
  - ❌ DON'T: Use nested includes without proper selection

- **Environment Setup**
  - ✅ DO: Include DATABASE_URL in .env
    ```env
    DATABASE_URL="postgresql://user:password@localhost:5432/db?schema=public"
    ```
  - ✅ DO: Include DIRECT_URL for serverless environments
  - ❌ DON'T: Commit database credentials to version control

- **File Organization**
  - ✅ DO: Keep Prisma schema in `prisma/schema.prisma`
  - ✅ DO: Keep client initialization in `src/lib/prisma.ts`
  - ✅ DO: Keep migrations in `prisma/migrations`
  - ❌ DON'T: Place Prisma-related files in component directories

- **Validation Rules**
  These rules will be automatically checked:
  1. All API routes using Prisma must have `runtime = 'nodejs'`
  2. No direct PrismaClient instantiation outside `src/lib/prisma.ts`
  3. Proper error handling around Prisma operations
  4. Consistent import pattern for Prisma client
  5. Transaction usage for multiple operations
  6. Proper field selection in queries

- **Automated Checks**
  The following patterns trigger warnings:
  - Missing runtime declaration in API routes
  - Direct PrismaClient imports
  - Missing try-catch blocks around Prisma operations
  - Missing transaction blocks for multiple operations
  - Missing field selection in queries

- **Migration and Deployment**
  - ✅ DO: Run migrations in deployment scripts
    ```bash
    npx prisma migrate deploy
    ```
  - ✅ DO: Generate Prisma client in build steps
    ```bash
    npx prisma generate
    ```
  - ❌ DON'T: Skip migration steps in deployment
  - ❌ DON'T: Deploy without generating client

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
