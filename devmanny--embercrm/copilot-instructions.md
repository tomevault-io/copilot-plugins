## embercrm

> Logging best practices

# Logging Rules

## Logger Import
```typescript
import { logger } from "@/lib/logger";
```

## Logging Pattern

**Object FIRST, message SECOND:**

```typescript
// CORRECT
logger.info({ userId, action }, "User performed action");
logger.error({ error, context }, "Operation failed");

// WRONG
logger.info("User performed action", { userId });
console.log("Debug:", data); // Never use console.log
```

## Log Levels

```typescript
logger.debug({ data }, "Detailed debug info");     // Development only
logger.info({ userId }, "Normal operation");       // General events
logger.warn({ issue }, "Potential problem");       // Warnings
logger.error({ error }, "Operation failed");       // Errors
logger.fatal({ error }, "Critical failure");       // Critical errors
```

## Structured Context

Include relevant context in log objects:

```typescript
// tRPC procedure
logger.info({
  userId: ctx.user.id,
  organizationId: ctx.organization.id,
  action: "lead.create",
  leadId: lead.id,
}, "Lead created");

// Error logging
logger.error({
  error,
  userId: ctx.user.id,
  input,
}, "Failed to create lead");
```

## Best Practices

- Never use `console.log` in production code
- Always include user context when available
- Log both start and completion of important operations
- Include error objects in error logs
- Use meaningful, searchable messages
- Don't log sensitive data (passwords, tokens, etc.)

## Example Pattern

```typescript
async function processOrder(orderId: string, ctx: Context) {
  const logContext = {
    orderId,
    userId: ctx.user.id,
    organizationId: ctx.organization.id,
  };

  logger.info(logContext, "Processing order started");

  try {
    const result = await doSomething();
    logger.info({ ...logContext, result }, "Order processed successfully");
    return result;
  } catch (error) {
    logger.error({ ...logContext, error }, "Order processing failed");
    throw error;
  }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devmanny) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
