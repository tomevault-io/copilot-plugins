## cc3

> Best practices for using Prisma ORM in the project


# Prisma Best Practices

Rule for ensuring consistent and efficient use of Prisma ORM.

<rule>
name: prisma_best_practices
description: Ensures Prisma schema and client usage follow best practices and conventions
filters:
  - type: file_extension
    pattern: "\\.prisma$"
  - type: content
    pattern: "prisma"

actions:
  - type: validate
    conditions:
      # Check for proper model naming
      - pattern: "model\\s+[A-Z][a-zA-Z]*\\s*{"
        message: "Model names must be PascalCase"

      # Check for proper field naming
      - pattern: "\\s+[a-z][a-zA-Z]*\\s+"
        message: "Field names must be camelCase"

      # Check for proper relation naming
      - pattern: "@relation\\([^)]+\\)"
        message: "Relations must be properly defined with names and fields"

  - type: suggest
    message: |
      Follow these Prisma best practices:

      1. Schema Design:
         - Use meaningful model and field names
         - Include proper indexes for frequently queried fields
         - Define explicit relations between models
         - Use appropriate field types and modifiers

      2. Client Usage:
         - Use transactions for related operations
         - Implement proper error handling
         - Optimize queries using include/select
         - Use middleware for cross-cutting concerns

      3. Performance:
         - Add indexes for frequently queried fields
         - Use batch operations when possible
         - Implement proper connection pooling
         - Monitor query performance

examples:
  - input: |
      model User {
        id Int @id @default(autoincrement())
        email String @unique
        posts Post[]
      }
    output: "Valid Prisma model definition"

  - input: |
      model post {
        Id Int @id
      }
    output: "Invalid: Model name should be PascalCase"

metadata:
  priority: high
  version: 1.0.0
</rule>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michaelbarone)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/michaelbarone)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
