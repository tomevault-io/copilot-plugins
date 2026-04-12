## booking-platform-web

> Code conventions and style standards


# Code Conventions

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `UserCard.tsx` |
| Hooks | camelCase + use | `useAuth.ts` |
| Functions | camelCase | `formatDate()` |
| Constants | SCREAMING_SNAKE | `API_BASE_URL` |
| Types/Interfaces | PascalCase | `UserDTO` |
| Enums | PascalCase | `UserRole` |
| Util files | kebab-case | `date-helpers.ts` |
| CSS classes | kebab-case | `user-card` |

## Component Structure

```tsx
// 1. Imports
import { useState } from 'react';

// 2. Types/Interfaces
interface Props {
  title: string;
  onClose?: () => void;
}

// 3. Local constants
const DEFAULT_TITLE = 'Untitled';

// 4. Component
export function MyComponent({ title, onClose }: Props) {
  // 4.1 Hooks (order: state, effects, custom hooks)
  const [isOpen, setIsOpen] = useState(false);
  
  // 4.2 Handlers
  const handleClick = () => {
    setIsOpen(true);
  };
  
  // 4.3 Early returns
  if (!title) return null;
  
  // 4.4 Render
  return (
    <div>
      <h1>{title}</h1>
    </div>
  );
}
```

## TypeScript

```tsx
// ✅ GOOD: Explicit types for props and public returns
function calculateTotal(items: Item[]): number {
  return items.reduce((sum, item) => sum + item.price, 0);
}

// ✅ GOOD: Inference for local variables
const total = calculateTotal(items); // TypeScript infers number

// ❌ BAD: any
const data: any = fetchData();

// ✅ GOOD: unknown + type guard
const data: unknown = fetchData();
if (isUserDTO(data)) {
  // data is UserDTO here
}
```

## Async/Await

```tsx
// ✅ GOOD: async/await with try-catch
async function fetchUser(id: string): Promise<User> {
  try {
    const response = await api.get(`/users/${id}`);
    return response.data;
  } catch (error) {
    throw new ApiError('Failed to fetch user', { cause: error });
  }
}

// ❌ BAD: nested .then().catch()
api.get('/users').then(res => res.data.map(u => ...)).catch(...)
```

## Comments

```tsx
// ✅ GOOD: Explain the "why", not the "what"
// Using setTimeout to avoid race condition with state
setTimeout(() => setIsVisible(true), 0);

// ❌ BAD: Comments the obvious
// Increment counter
counter++;
```

## Props Destructuring

```tsx
// ✅ GOOD: Destructuring in parameters
function UserCard({ name, email, avatar }: UserCardProps) {

// ❌ BAD: Access via props.
function UserCard(props: UserCardProps) {
  return <div>{props.name}</div>;
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/santiagoavilez)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/santiagoavilez)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
