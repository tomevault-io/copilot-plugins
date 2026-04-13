## bmr-breegade

> **MUST** use Trigger.dev for event processing and background tasks.


# Events Rules

## Event Processing Framework

**MUST** use Trigger.dev for event processing and background tasks.

**Event Definition Pattern:**

```typescript
// src/services/trigger/events/UserCreatedEvent.ts
import { logger, task } from "@trigger.dev/sdk";
import User from "@/domain/entities/User";

export const UserCreatedEvent = task({
  id: "UserCreatedEvent",
  maxDuration: 300, // Stop after 300 secs (5 mins)
  run: async (payload: Partial<User>, { ctx }) => {
    logger.log("Processing user creation", { payload, ctx });

    // Event processing logic here
    await processUserCreation(payload);
  },
});

export default UserCreatedEvent;
```

## Event Organization

**MUST** place event definitions in `src/services/trigger/events/` directory.

**MUST** use PascalCase for event names ending with "Event":

- `UserCreatedEvent.ts`
- `OrderProcessedEvent.ts`
- `PaymentCompletedEvent.ts`

## Event Structure Requirements

**MUST** include these properties in every event:

```typescript
export const EventName = task({
  id: "EventName", // Unique identifier matching the export name
  maxDuration: 300, // Timeout in seconds
  run: async (payload: PayloadType, { ctx }) => {
    // Event processing logic
  },
});
```

**MUST** use typed payload parameters based on domain entities:

```typescript
run: async (payload: Partial<User>, { ctx }) => {
  // payload is properly typed
};
```

## Event Payload Rules

**MUST** use domain entity types or Partial<Entity> for payloads:

```typescript
// Good: Uses domain entity
run: async (payload: Partial<User>, { ctx }) => { ... }

// Good: Uses specific type
interface OrderPayload {
  orderId: string;
  userId: string;
  amount: number;
}
run: async (payload: OrderPayload, { ctx }) => { ... }
```

**MUST NOT** use `any` or overly generic types for payloads.

## Logging Requirements

**MUST** use Trigger.dev logger for all event logging:

```typescript
import { logger } from "@trigger.dev/sdk";

// Log event start
logger.log("Event started", { payload, ctx });

// Log progress
logger.info("Processing step completed", { step: "validation" });

// Log errors
logger.error("Event failed", { error: error.message });

// Log completion
logger.log("Event completed successfully", { result });
```

## Error Handling in Events

**MUST** handle errors gracefully in event processors:

```typescript
run: async (payload: Partial<User>, { ctx }) => {
  try {
    await processUserCreation(payload);
    logger.log("User creation processed successfully");
  } catch (error) {
    logger.error("Failed to process user creation", {
      error: error.message,
      userId: payload.id,
    });
    throw error; // Re-throw to trigger retry mechanism
  }
};
```

## Event Triggers

**MUST** trigger events from service layer, not directly from controllers:

```typescript
// src/services/auth/AuthService.ts
import UserCreatedEvent from "@/services/trigger/events/UserCreatedEvent";

class AuthService {
  async createUser(userData: Partial<User>): Promise<User> {
    const user = await userRepository.create(userData);

    // Trigger event after successful creation
    await UserCreatedEvent.trigger(user);

    return user;
  }
}
```

## Integration with External Services

**MUST** handle external service integrations within event handlers:

```typescript
// Integration with email service
export const UserCreatedEvent = task({
  id: "UserCreatedEvent",
  maxDuration: 300,
  run: async (payload: Partial<User>, { ctx }) => {
    logger.log("Sending welcome email", { userId: payload.id });

    const { data, error } = await resend.emails.send({
      from: "no-reply@yourapp.com",
      to: [payload.email!],
      subject: "Welcome to our platform",
      html: `<h1>Welcome ${payload.firstName}!</h1>`,
    });

    if (error) {
      logger.error("Failed to send welcome email", {
        error,
        userId: payload.id,
      });
      throw new Error(`Email sending failed: ${error.message}`);
    }

    logger.log("Welcome email sent successfully", {
      messageId: data?.id,
      userId: payload.id,
    });
  },
});
```

## Event Configuration

**MUST** configure Trigger.dev in project configuration:

```typescript
// trigger.config.ts
import { defineConfig } from "@trigger.dev/sdk";

export default defineConfig({
  project: "your-project-id",
  // other configuration
});
```

**MUST** include trigger script in package.json:

```json
{
  "scripts": {
    "trigger": "npx trigger.dev@latest dev"
  }
}
```

## Event Naming Conventions

**Event IDs MUST** match the export name:

```typescript
export const UserCreatedEvent = task({
  id: "UserCreatedEvent", // Must match export name
  // ...
});
```

**Event files MUST** use PascalCase with "Event" suffix:

- File: `UserCreatedEvent.ts`
- Export: `UserCreatedEvent`
- ID: `"UserCreatedEvent"`

## Event Dependencies

**MUST** import required services and repositories at the top:

```typescript
import { logger, task } from "@trigger.dev/sdk";
import User from "@/domain/entities/User";
import resend from "@/services/resend";
import db from "@/persistance/db";
```

**MUST** avoid circular dependencies between events and services.

## Event Timeouts

**MUST** set appropriate `maxDuration` based on event complexity:

- Simple events: 60-300 seconds
- Complex processing: 300-900 seconds
- Long-running tasks: 900+ seconds

**Example timeout guidelines:**

```typescript
// Email sending
maxDuration: 60,

// Data processing
maxDuration: 300,

// File uploads/downloads
maxDuration: 900,
```

## Event Testing

**MUST** create testable event handlers by extracting logic:

```typescript
// Testable business logic
export const processUserCreation = async (user: Partial<User>) => {
  // Business logic here
  return await sendWelcomeEmail(user);
};

// Event handler delegates to business logic
export const UserCreatedEvent = task({
  id: "UserCreatedEvent",
  maxDuration: 300,
  run: async (payload: Partial<User>, { ctx }) => {
    logger.log("Processing user creation", { payload });
    return await processUserCreation(payload);
  },
});
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bmarguerite) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
