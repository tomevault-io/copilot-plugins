## docmcp

> Prisma Instance Injection Pattern for Integration tests


 Prisma Instance Injection Pattern for Integration tests
 
 1. All service classes should accept an optional PrismaClient parameter in their constructor
 2. Use the main Prisma client as a fallback when no client is provided
 3. Store the Prisma instance as a private property

 Example:
 ```typescript
 constructor(prismaClient?: PrismaClient) {
   this.prisma = prismaClient || getMainPrismaClient();
 }
 ```

 This pattern allows for:
 - Using the standard client in normal operation
 - Injecting test database clients during testing
 - Easier mocking for unit tests
 - Better isolated tests to prevent test data cross-contamination

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/visheshd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
