## astral-assesment

> **Core Principle:** Write TypeScript and React code with absolute clarity as the top priority. Choose the most straightforward solution that any developer can understand at first glance.

# TypeScript & TSX Guidelines - v2

**Core Principle:** Write TypeScript and React code with absolute clarity as the top priority. Choose the most straightforward solution that any developer can understand at first glance.

**Development Philosophy:**

1. **Fewest lines, straightforward over clever** → Build only when needed
2. **One responsibility per function/component/module** → Single clear purpose
3. **Five-minute rule** → If you cannot explain it quickly, it is too complex
4. **Write for humans first** → Clever code is bad code

**Simple means:**

- Understandable in under 30 seconds
- Clear names: `calculateMonthlyPayment()` not `calc()`
- Max three nesting levels → Prefer early returns
- Numbered comments for multi-step flows: `// 1️⃣ Validate → // 2️⃣ Process → // 3️⃣ Return`

## Style Conventions

**TypeScript with these specifics:**

- Double quotes for strings
- Trailing commas in multi-line structures
- Type hints everywhere
- **Zod for all runtime validation** - Use schemas for uncertain data
- **Favor functions over classes** - Classes only for true stateful objects

**Naming Conventions:**

- `camelCase` for functions, variables, props
- `PascalCase` for components, types, interfaces
- `SCREAMING_SNAKE_CASE` for constants
- Prefix boolean props: `isLoading`, `hasError`, `canEdit`, `shouldShow`

```typescript
function createUserProfile(userData: CreateUserRequest, settings?: UserSettings, notifyAdmin: boolean = true): Promise<UserProfile> {
  // Implementation
}

const MAX_FILE_SIZE_MB = 10;
const ALLOWED_EXTENSIONS = [".jpg", ".png", ".gif", ".pdf"];

// Zod for runtime validation of uncertain inputs
const UserCreateRequestSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
  age: z.number().min(13).max(120),
});

type UserCreateRequest = z.infer<typeof UserCreateRequestSchema>;
```

## File Structure

**1. Header (Required)**

```typescript
/* ==========================================================================*/
// userService.ts — User management and authentication operations
/* ==========================================================================*/
// Purpose: Handle user CRUD operations and session management
// Sections: Imports, Types, Functions, Public API
/* ==========================================================================*/
```

**2. Imports (Four groups)**

```typescript
/* ==========================================================================*/
// Imports
/* ==========================================================================*/

// Standard Library/Runtime --------------------------------------------------
import fs from "fs";
import path from "path";
import { EventEmitter } from "events";

// Third Party ---------------------------------------------------------------
import { z } from "zod";
import axios from "axios";
import express from "express";

// Core (App-wide) -----------------------------------------------------------
import { DatabaseConnection } from "@/lib/database";
import { logger } from "@/lib/logger";
import { AppError } from "@/lib/errors";
import { User, UserProfile } from "@/types/user";

// Internal (Current Module) -------------------------------------------------
import { validateEmail } from "./helpers";
import { formatUserData } from "./utils";
```

**3. Types (Component-specific only)**

```typescript
/* ==========================================================================*/
// Types (Component-specific only)
/* ==========================================================================*/

// Only include types that are hyper-specific to this file
interface UserProfileProps {
  userId: string;
  editable?: boolean;
  onUpdate?: (user: User) => void;
}

// App-wide types should be in dedicated @/types files
```

**4. Public API - Be Intentional**

```typescript
/* ==========================================================================*/
// Public API
/* ==========================================================================*/

// Don't re-export everything - be intentional about what's public
export { createUser, getUserById }; // Only these are part of public API
export type { CreateUserRequest }; // Only export types others need

// Keep internal - don't export helpers, utilities, or implementation details
// validateEmail, formatUserData, etc. stay private to this module
```

## Error Handling - Fail Fast & Simple

**Core Principles:**

1. **Validate only uncertain inputs** - API responses, user input, external data
2. **Don't re-validate TypeScript-guaranteed values** - Trust your types
3. **Catch only what you can handle** - Let other exceptions bubble up
4. **One try/catch per function max** - If you need more, refactor
5. **Structured logging with context** - Preserve error causes and original stack traces
6. **Consider Result types for predictable failures** - Reserve exceptions for truly exceptional cases

```typescript
// Custom error class that preserves cause
class AppError extends Error {
  constructor(message: string, options?: { cause?: Error; userId?: string; context?: Record<string, unknown> }) {
    super(message, { cause: options?.cause });
    this.name = "AppError";
  }
}

// Result type for predictable failures in hot paths
type Result<T, E = Error> = { success: true; data: T } | { success: false; error: E };

async function processPayment(paymentData: unknown, userId: string): Promise<PaymentResult> {
  // Step 1: Validate uncertain input ----
  const validatedData = PaymentRequestSchema.parse(paymentData);

  if (!userId.trim()) {
    throw new Error("User ID cannot be empty");
  }

  // Step 2: Process payment ----
  try {
    const result = await paymentGateway.charge(validatedData);
    await updateUserBalance(userId, -validatedData.amount);
    return { success: true, transactionId: result.id };
  } catch (err) {
    // Structured logging with context
    logger.error({ userId, amount: validatedData.amount, err }, "Payment failed");
    // Preserve original error cause
    throw new AppError("Payment processing failed", { cause: err, userId });
  }
}

// Result type for predictable failures
async function fetchUserSafely(userId: string): Promise<Result<User>> {
  if (!userId.trim()) {
    return { success: false, error: new Error("User ID cannot be empty") };
  }

  try {
    const user = await getUserById(userId);
    return { success: true, data: user };
  } catch (err) {
    logger.warn({ userId, err }, "User fetch failed");
    return { success: false, error: err as Error };
  }
}

function validateUserRegistration(data: unknown): CreateUserRequest {
  try {
    return UserCreateRequestSchema.parse(data);
  } catch (err) {
    // Structured logging
    logger.warning({ data, err }, "Invalid registration data");
    // Preserve cause in re-thrown error
    throw new AppError("Registration validation failed", { cause: err });
  }
}
```

## Function-First Design

**Favor functions over classes:**

```typescript
// Step 1: Validate input ----
async function createUser(userData: unknown): Promise<User> {
  const validatedData = UserCreateRequestSchema.parse(userData);

  const existingUser = await getUserByEmail(validatedData.email);
  if (existingUser) {
    throw new Error(`User with email ${validatedData.email} already exists`);
  }

  // Step 2: Create user record ----
  const user: User = {
    id: generateId(),
    email: validatedData.email,
    name: validatedData.name,
    createdAt: new Date(),
    isActive: true,
  };

  // Step 3: Save to database ----
  await saveUser(user);
  return user;
}

async function getUserById(userId: string): Promise<User | null> {
  if (!userId.trim()) {
    throw new Error("User ID cannot be empty");
  }

  return await db.findById(userId);
}
```

## React Component Patterns

**Simple functional components with proper prop destructuring:**

```tsx
interface UserCardProps {
  user: User;
  isSelected?: boolean;
  onSelect?: (userId: string) => void;
}

function UserCard({ user, isSelected = false, onSelect }: UserCardProps) {
  if (!user) return null;

  return (
    <div>
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      {onSelect && <button onClick={() => onSelect(user.id)}>{isSelected ? "Selected" : "Select"}</button>}
    </div>
  );
}

function UserProfile({ userId, editable = false }: UserProfileProps) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    if (!userId) {
      setError("User ID is required");
      setIsLoading(false);
      return;
    }

    loadUser();
  }, [userId]);

  const loadUser = async () => {
    try {
      setIsLoading(true);
      setError(null);

      const userData = await getUserById(userId);
      if (!userData) {
        throw new Error("User not found");
      }

      setUser(userData);
    } catch (err) {
      setError(err instanceof Error ? err.message : "Unknown error");
    } finally {
      setIsLoading(false);
    }
  };

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      {editable && <button onClick={() => console.log("Edit user")}>Edit</button>}
    </div>
  );
}
```

## Custom Hooks

**Simple, focused custom hooks:**

```tsx
function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    if (!userId) {
      setError("User ID is required");
      setIsLoading(false);
      return;
    }

    const loadUser = async () => {
      try {
        setIsLoading(true);
        setError(null);
        const userData = await getUserById(userId);
        setUser(userData);
      } catch (err) {
        setError(err instanceof Error ? err.message : "Unknown error");
      } finally {
        setIsLoading(false);
      }
    };

    loadUser();
  }, [userId]);

  return { user, isLoading, error };
}
```

## Performance - React 19 Handles Most

**React 19 automatically optimizes most cases - be selective with manual optimization:**

```tsx
// React 19 handles this automatically - no useMemo needed
function UserList({ users, searchTerm }: UserListProps) {
  const filteredUsers = users.filter((user) => user.name.toLowerCase().includes(searchTerm.toLowerCase()));

  const handleEdit = (user: User) => {
    editUser(user);
  };

  return (
    <div>
      {filteredUsers.map((user) => (
        <UserCard key={user.id} user={user} onEdit={handleEdit} />
      ))}
    </div>
  );
}

// Only optimize when you measure actual performance problems
function ExpensiveComponent({ data }: ExpensiveProps) {
  // Only use useMemo if you have proven performance issues
  const expensiveResult = useMemo(() => {
    return reallyExpensiveOperation(data);
  }, [data]);

  return <div>{expensiveResult}</div>;
}
```

## Async Patterns

**Simple concurrent operations:**

```typescript
async function fetchUserDashboard(userId: string): Promise<UserDashboard> {
  if (!userId.trim()) {
    throw new Error("User ID is required");
  }

  const [user, projects, notifications] = await Promise.all([getUserById(userId), getUserProjects(userId), getUserNotifications(userId)]);

  return { user, projects, notifications };
}
```

## Resource Management

**Clean up resources properly:**

```tsx
function UserNotifications({ userId }: UserNotificationsProps) {
  const [notifications, setNotifications] = useState<Notification[]>([]);

  useEffect(() => {
    if (!userId) return;

    const controller = new AbortController();

    const loadNotifications = async () => {
      try {
        const data = await fetchNotifications(userId, {
          signal: controller.signal,
        });
        setNotifications(data);
      } catch (error) {
        if (error.name !== "AbortError") {
          console.error("Failed to load notifications:", error);
        }
      }
    };

    loadNotifications();

    return () => {
      controller.abort();
    };
  }, [userId]);

  return (
    <div>
      {notifications.map((notification) => (
        <div key={notification.id}>{notification.message}</div>
      ))}
    </div>
  );
}
```

## Type Organization & Export Strategy

**Be intentional about what you export - don't re-export everything:**

```typescript
// users/types.ts - Dedicated types file for app-wide types
export interface User {
  id: string;
  email: string;
  name: string;
  createdAt: Date;
  isActive: boolean;
}

export interface CreateUserRequest {
  email: string;
  name: string;
  password: string;
}

// users/service.ts - Implementation file
/* ==========================================================================*/
// Types (Component-specific only)
/* ==========================================================================*/

// Only hyper-specific types that won't be used elsewhere
interface ProcessingOptions {
  validateEmail: boolean;
  sendWelcomeEmail: boolean;
}

/* ==========================================================================*/
// Public API - Be Intentional
/* ==========================================================================*/

// Only export what other modules actually need
export { createUser, getUserById };
export type { CreateUserRequest };

// Keep these internal - don't export helpers or implementation details:
// - validateUserData (helper function)
// - ProcessingOptions (internal type)
// - formatUserResponse (utility function)
```

**Component files - keep prop interfaces local:**

```tsx
// UserProfile.tsx
import { User } from "@/types/user";

// Hyper-specific to this component - keep local
interface UserProfileProps {
  user: User;
  isEditable?: boolean;
  onSave?: (user: User) => void;
}

function UserProfile({ user, isEditable = false, onSave }: UserProfileProps) {
  // Implementation
}

// Only export the component - prop interface stays private
export { UserProfile };
```

**Index files - intentional barrel exports only:**

```typescript
// users/index.ts - Clean public API only
export { createUser, getUserById, deleteUser } from "./service";
export { UserProfile, UserCard } from "./components";
export type { User, CreateUserRequest } from "./types";

// Don't export:
// - Helper functions from service
// - Internal component props
// - Utility functions
// - Implementation details
```

---

**Remember**:

- **Function-first design** - Avoid classes unless truly needed
- **Zod for uncertain input validation** - Trust TypeScript for internal types
- **React 19 handles most optimizations** - Only optimize when you measure problems
- **Fail fast on uncertain inputs** - Don't over-validate guaranteed values
- **Be intentional about exports** - Don't re-export everything, keep implementation details private
- **Dedicated types files** - Component prop interfaces stay local, shared types go in dedicated files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arnaavsareen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
