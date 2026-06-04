## eslint

> - All code must follow **ESLint** rules for consistency and quality


# 🔧 ESLint Rules & Code Quality

## General Principles

- All code must follow **ESLint** rules for consistency and quality
- Use **Prettier** for code formatting (integrated with ESLint)
- Follow **JSDoc** standards for documentation
- Maintain **import sorting** and organization

## ESLint Configuration

### Import Sorting Rules

- **External libraries** first (React, Next.js, etc.)
- **Internal modules** second (@/shared, @/modules)
- **Relative imports** last (./, ../)
- **Type imports** should be separate from value imports

```typescript
// ✅ Correct import order
import React from "react";
import { NextRequest } from "next/server";
import { z } from "zod";

import { Button } from "@/shared/ui";
import { useUser } from "@/modules/users/hooks";

import { UserCard } from "./UserCard";
import type { UserCardProps } from "./types";
```

### JSDoc Documentation Rules

**REQUIRED JSDoc for:**

- **Public functions** and **hooks**
- **Complex utility functions**
- **Component props interfaces**
- **API endpoints** and **data fetching functions**

**JSDoc Format:**

````typescript
/**
 * Brief description of the function
 *
 * @param paramName - Description of parameter
 * @param paramName - Description of parameter
 * @returns Description of return value
 * @throws {ErrorType} When this error occurs
 * @example
 * ```typescript
 * const result = functionName(param1, param2);
 * ```
 */
````

### Code Quality Rules

- **No unused variables** or imports
- **Consistent naming** (camelCase for functions, PascalCase for components)
- **Proper TypeScript types** (avoid `any`, use specific types)
- **Error handling** for async operations
- **Accessibility** attributes for UI components

## File Organization

### Import Structure

```typescript
// 1. React and external libraries
import React from "react";
import { useState, useEffect } from "react";

// 2. Next.js imports
import { NextRequest } from "next/server";

// 3. Third-party libraries
import { z } from "zod";
import { toast } from "react-toastify";

// 4. Internal shared modules
import { Button } from "@/shared/ui";
import { useCustomMutation } from "@/shared/lib/client";

// 5. Internal module imports
import { useUser } from "@/modules/users/hooks";
import { User } from "@/modules/users/schemas";

// 6. Relative imports
import { UserCard } from "./UserCard";
import { UserAvatar } from "../UserAvatar/UserAvatar";

// 7. Type imports (separate)
import type { UserCardProps } from "./types";
import type { ComponentProps } from "react";
```

### Function Documentation

````typescript
/**
 * Formats user display name from user object
 *
 * @param user - User object containing name and email
 * @returns Formatted display name or email prefix
 * @example
 * ```typescript
 * const displayName = getUserDisplayName({ name: 'John Doe', email: 'john@example.com' });
 * // Returns: 'John Doe'
 * ```
 */
export const getUserDisplayName = (user: User): string => {
  return user.name || user.email.split("@")[0];
};
````

### Component Documentation

```typescript
/**
 * UserCard component for displaying user information
 *
 * @param user - User object to display
 * @param showActions - Whether to show action buttons
 * @param onEdit - Callback when edit button is clicked
 * @param onDelete - Callback when delete button is clicked
 * @param className - Additional CSS classes
 */
export interface UserCardProps {
  user: User;
  showActions?: boolean;
  onEdit?: (user: User) => void;
  onDelete?: (user: User) => void;
  className?: string;
}
```

## Error Handling

- **Always handle errors** in async operations
- **Use proper error types** instead of generic Error
- **Log errors** appropriately (console.error for development)
- **Provide user-friendly error messages**

## Performance Rules

- **Use React.memo** for expensive components
- **Use useMemo/useCallback** for expensive calculations
- **Avoid inline object/function creation** in render
- **Lazy load** heavy components when possible

## Accessibility Rules

- **Use semantic HTML** elements
- **Add ARIA labels** where needed
- **Ensure keyboard navigation** works
- **Provide alt text** for images
- **Use proper heading hierarchy** (h1, h2, h3...)

---
> Source: [Adil78124/AgroHub](https://github.com/Adil78124/AgroHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
