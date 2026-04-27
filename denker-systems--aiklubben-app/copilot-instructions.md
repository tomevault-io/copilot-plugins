## 08-code-quality-rules

> - **Mode**: Always On

# Code Quality Rules - TypeScript & React Native

## Activation

- **Mode**: Always On
- **Description**: Code quality standards and TypeScript best practices

---

## TypeScript Configuration

### Strict Mode Requirements

```json
// tsconfig.json - REQUIRED settings
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "useUnknownInCatchVariables": true,
    "alwaysStrict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  }
}
```

---

## Type Definitions

### NEVER Use `any`

```typescript
// WRONG: Using any
const processData = (data: any) => {};
const items: any[] = [];

// CORRECT: Use specific types
const processData = (data: UserData) => {};
const items: User[] = [];

// CORRECT: Use unknown for truly unknown data
const parseJSON = (json: string): unknown => JSON.parse(json);

// CORRECT: Use generics for flexible types
const processArray = <T>(items: T[]): T[] => items;
```

### Type vs Interface

```typescript
// USE interface for:
// - Object shapes that may be extended
// - Component props
// - API responses

interface UserProps {
  name: string;
  email: string;
}

interface AdminProps extends UserProps {
  permissions: string[];
}

// USE type for:
// - Union types
// - Intersection types
// - Mapped types
// - Utility types

type Status = 'idle' | 'loading' | 'success' | 'error';
type Nullable<T> = T | null;
type UserOrAdmin = User | Admin;
```

### Type Assertions

```typescript
// AVOID type assertions when possible
// WRONG: Unsafe assertion
const user = data as User;

// CORRECT: Type guard
const isUser = (data: unknown): data is User => {
  return typeof data === 'object' && data !== null && 'name' in data && 'email' in data;
};

if (isUser(data)) {
  // data is now typed as User
  console.log(data.name);
}

// ACCEPTABLE: Assertion with validation
const element = document.getElementById('root') as HTMLElement | null;
if (!element) throw new Error('Root element not found');
```

---

## Null Safety

### Defensive Null Checks

```typescript
// ALWAYS check for null/undefined before use
// WRONG: Unsafe access
const name = user.profile.name;

// CORRECT: Optional chaining
const name = user?.profile?.name;

// CORRECT: Nullish coalescing for defaults
const name = user?.profile?.name ?? 'Unknown';

// CORRECT: Type narrowing
if (user && user.profile) {
  const name = user.profile.name;
}
```

### Non-Null Assertion Usage

```typescript
// AVOID non-null assertion (!) when possible
// WRONG: Hiding potential null issues
const value = possiblyNull!.property;

// CORRECT: Explicit check
if (possiblyNull) {
  const value = possiblyNull.property;
}

// ACCEPTABLE: After explicit null check
const items = data?.items;
if (!items) return null;
const firstItem = items[0]; // Safe after check
```

---

## Function Patterns

### Early Returns

```typescript
// CORRECT: Early returns reduce nesting
const processUser = (user: User | null): Result => {
  if (!user) {
    return { error: 'No user provided' };
  }

  if (!user.isActive) {
    return { error: 'User is not active' };
  }

  if (!user.hasPermission) {
    return { error: 'User lacks permission' };
  }

  // Main logic with all checks passed
  return { data: process(user) };
};

// WRONG: Deep nesting
const processUserBad = (user: User | null): Result => {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        return { data: process(user) };
      } else {
        return { error: 'User lacks permission' };
      }
    } else {
      return { error: 'User is not active' };
    }
  } else {
    return { error: 'No user provided' };
  }
};
```

### Function Parameter Limits

```typescript
// WRONG: Too many parameters
const createUser = (
  name: string,
  email: string,
  age: number,
  role: string,
  department: string,
  startDate: Date,
) => {};

// CORRECT: Use options object for 3+ parameters
interface CreateUserOptions {
  name: string;
  email: string;
  age?: number;
  role?: string;
  department?: string;
  startDate?: Date;
}

const createUser = (options: CreateUserOptions) => {};
```

---

## Async/Await Patterns

### Error Handling

```typescript
// CORRECT: Explicit try-catch with typed errors
const fetchUser = async (id: string): Promise<User | null> => {
  try {
    const response = await api.get(`/users/${id}`);
    return response.data;
  } catch (error) {
    if (error instanceof ApiError) {
      console.error('API Error:', error.message);
    } else if (error instanceof Error) {
      console.error('Error:', error.message);
    }
    return null;
  }
};

// CORRECT: Result type pattern
type Result<T, E = Error> = { success: true; data: T } | { success: false; error: E };

const fetchUserResult = async (id: string): Promise<Result<User>> => {
  try {
    const response = await api.get(`/users/${id}`);
    return { success: true, data: response.data };
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error : new Error('Unknown error'),
    };
  }
};
```

### Avoid Floating Promises

```typescript
// WRONG: Floating promise
useEffect(() => {
  fetchData(); // Promise not handled
}, []);

// CORRECT: Handle promise
useEffect(() => {
  const load = async () => {
    try {
      await fetchData();
    } catch (error) {
      console.error(error);
    }
  };
  load();
}, []);

// CORRECT: Using .then/.catch
useEffect(() => {
  fetchData().then(setData).catch(console.error);
}, []);
```

---

## Import Organization

### Import Order (Enforced)

```typescript
// 1. React imports
import React, { useState, useEffect, useCallback } from 'react';

// 2. React Native imports
import { View, Text, StyleSheet, Pressable } from 'react-native';

// 3. Third-party libraries (alphabetical)
import { MotiView } from 'moti';
import * as Haptics from 'expo-haptics';

// 4. Navigation
import { useNavigation } from '@react-navigation/native';

// 5. Local components (absolute imports)
import { Button, Card, Text as UIText } from '@/components/ui';
import { ScreenLayout } from '@/components/layout';

// 6. Hooks
import { useAuth } from '@/hooks/useAuth';

// 7. Utils/helpers
import { formatDate } from '@/lib/utils';

// 8. Types
import type { User, Course } from '@/types';

// 9. Constants
import { COLORS, SPACING } from '@/constants';
```

### Absolute Imports

```typescript
// CORRECT: Use absolute imports with @ alias
import { Button } from '@/components/ui';
import { useAuth } from '@/hooks/useAuth';
import type { User } from '@/types';

// WRONG: Relative imports (except for local files)
import { Button } from '../../../components/ui';
import { useAuth } from '../../hooks/useAuth';

// EXCEPTION: Local file in same directory
import { HelperFunction } from './helpers';
```

---

## Naming Conventions

### File Naming

```
Components:     PascalCase.tsx     (Button.tsx, UserCard.tsx)
Hooks:          camelCase.ts       (useAuth.ts, useFetch.ts)
Utils:          camelCase.ts       (formatDate.ts, validation.ts)
Types:          camelCase.ts       (user.ts, navigation.ts)
Constants:      camelCase.ts       (colors.ts, theme.ts)
Screens:        PascalCase.tsx     (HomeScreen.tsx)
```

### Variable/Function Naming

```typescript
// Components: PascalCase
const UserProfile: React.FC = () => {};

// Hooks: camelCase starting with 'use'
const useUserData = () => {};

// Event handlers: handle + Event
const handlePress = () => {};
const handleSubmit = () => {};
const handleChange = (value: string) => {};

// Booleans: is/has/should/can prefix
const isLoading = true;
const hasError = false;
const shouldRefresh = true;
const canSubmit = form.isValid;

// Constants: SCREAMING_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = 'https://api.example.com';
```

---

## Comments

### When to Comment

```typescript
// COMMENT: Complex business logic
// Calculate discount based on user tier and purchase history
// Tier multipliers: Gold=0.8, Silver=0.9, Bronze=0.95
const calculateDiscount = (user: User, amount: number): number => {
  const tierMultiplier = TIER_MULTIPLIERS[user.tier];
  return amount * tierMultiplier;
};

// COMMENT: Non-obvious workarounds
// iOS requires explicit height for ScrollView inside flex container
// See: https://github.com/facebook/react-native/issues/XXX
<ScrollView style={{ flex: 1 }} />

// DON'T COMMENT: Obvious code
// WRONG:
// Set loading to true
setLoading(true);
```

---

## Forbidden Code Quality Practices

1. **NEVER** use `any` type - use `unknown` or proper types
2. **NEVER** use non-null assertion without prior validation
3. **NEVER** leave console.log in production code
4. **NEVER** use var - always use const or let
5. **NEVER** mutate function parameters
6. **NEVER** use == for comparison - always use ===
7. **NEVER** nest ternary operators more than once
8. **NEVER** exceed 3 levels of callback nesting

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
