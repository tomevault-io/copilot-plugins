## typescript-patterns

> This project uses strict TypeScript configuration with modern patterns and best practices.


# TypeScript Patterns

This project uses strict TypeScript configuration with modern patterns and best practices.

## Configuration

- **Strict Mode**: Enabled in [tsconfig.json](mdc:tsconfig.json)
- **Path Mapping**: `@/` alias for clean imports
- **Type Checking**: Strict type checking enabled

## Key Patterns

### Environment Configuration
```typescript
// Use Zod for environment validation
const envSchema = z.object({
  NEXT_PUBLIC_APP_NAME: z.string().default("Stacks Next Template"),
});
```

### Component Props
```typescript
// Use proper TypeScript for component props
interface ComponentProps {
  children: ReactNode;
  className?: string;
}
```

### Hook Return Types
```typescript
// Define explicit return types for hooks
export function useWallet() {
  return {
    data: { address: string; publicKey?: string } | null;
    isConnected: boolean;
    connect: () => Promise<[string, string]>;
    disconnect: () => void;
  };
}
```

## Type Safety Rules

1. **No `any` types** - Use proper typing or `unknown`
2. **Explicit returns** - Define return types for functions
3. **Interface over type** - Use interfaces for object shapes
4. **Generic constraints** - Use proper generic constraints
5. **Null safety** - Handle null/undefined cases explicitly

## Import Patterns

```typescript
// Prefer named imports
import { useWallet } from "@/hooks/wallet";
import { Button } from "@/components/ui/button";

// Use type-only imports when needed
import type { AppConfig } from "./client";
```

## Error Handling

```typescript
// Use proper error types
try {
  const result = await operation();
} catch (error) {
  if (error instanceof Error) {
    console.error(error.message);
  }
}
```

## Component Patterns

```typescript
// Use proper component typing
export function Component({ 
  children, 
  className 
}: {
  children: ReactNode;
  className?: string;
}) {
  return <div className={className}>{children}</div>;
}
```

---
> Source: [Mcsavvy/stacks-next-template](https://github.com/Mcsavvy/stacks-next-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
