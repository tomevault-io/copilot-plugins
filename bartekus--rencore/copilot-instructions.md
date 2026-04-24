## rencore

> Service-to-Service API Calls

# Service-to-Service API Calls

Standards for making service-to-service API calls in Encore.ts applications.

<rule>
name: api_calls
description: Enforces standards for service-to-service API communication
filters:
  # Match TypeScript files
  - type: file_extension
    pattern: "\\.ts$"
  # Match service imports
  - type: content
    pattern: "from\\s+['\"]~encore/clients['\"]"
  # Match file modifications
  - type: event
    pattern: "file_modify"

actions:
  - type: enforce
    conditions:
      # Enforce proper service imports
      - pattern: "require\\(['\"]~encore/clients['\"]\\)"
        message: "Use ES6 import syntax for service imports"

      # Enforce async/await usage
      - pattern: "\\.then\\("
        message: "Use async/await syntax instead of promises"

  - type: suggest
    message: |
      When making service-to-service API calls:

      1. Import Standards:
         - Import services from ~encore/clients
         - Use ES6 import syntax
         - Import specific endpoints needed

      2. Call Pattern:
         - Use async/await syntax
         - Treat API calls as regular async functions
         - Maintain type safety in responses

      3. Best Practices:
         - Handle errors appropriately
         - Utilize IDE autocompletion
         - Leverage compile-time type checking

      4. Example Structure:
         ```typescript
         import { serviceA } from "~encore/clients";

         export const myAPI = api({}, async (): Promise<void> => {
           try {
             const response = await serviceA.endpoint({ param: "value" });
             // Handle response
           } catch (error) {
             // Handle error
           }
         });
         ```

      5. Database Operations:
         ```typescript
         // Database connection
         const db = drizzle(DB.connectionString);

         // Query patterns
         const result = await db.select()
           .from(users)
           .where(eq(users.id, id))
           .limit(1);

         // Pagination
         const offset = getOffset(page, limit);
         const results = await db.select()
           .from(users)
           .orderBy(asc(users.id))
           .limit(limit)
           .offset(offset);
         ```

examples:
  - input: |
      # Bad: Promise chains
      serviceA.endpoint()
        .then(response => {
          // Handle response
        });

      # Good: Async/await
      import { hello } from "~encore/clients";

      export const myOtherAPI = api({}, async (): Promise<void> => {
        const resp = await hello.ping({ name: "World" });
        console.log(resp.message);
      });
    output: "Proper service-to-service API call implementation"

metadata:
  priority: high
  version: 1.0
</rule>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bartekus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
