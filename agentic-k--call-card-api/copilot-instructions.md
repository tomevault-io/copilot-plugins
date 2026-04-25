## call-card-api

> Standards for generating, updating, and maintaining Supabase Edge Function clients consistently across projects.

# Supabase Function Invocation Standards

## Description
Standards for generating, updating, and maintaining Supabase Edge Function clients consistently across projects.

## Globs
*.ts

## Rules

### File Structure
- Core helper must be in `src/lib/supabase/functionsClient.ts`
- Domain modules must be in parallel folders (e.g., `src/lib/supabase/templatesClient.ts`)
- Group by feature or bounded context to keep files small and focused

### Naming Conventions
- File names must use `camelCase` (e.g., `functionsClient.ts`)
- Function names must follow these patterns:
  - Generic CRUD wrappers: `getFunction`, `postFunction`, `putFunction`, `patchFunction`, `deleteFunction`
  - Domain-specific calls: `<verb><Resource>` (e.g., `getTemplates`, `createUser`)

### Import Statements
```typescript
import { supabase } from '@/integrations/supabase/client'
```
- Only import types (e.g., `FunctionInvokeOptions`, `Database`) where needed

### Core Invoker Helper
```typescript
async function invokeFn<T>(
  name: string,
  opts: FunctionInvokeOptions
): Promise<T> { … }
```

Must include:
1. Authorization: Merge in `Authorization: Bearer <token>` from `supabase.auth.getSession()`
2. URL handling: Map `name` one-to-one with Edge Function path
3. JSON serialization: Let `supabase-js` stringify objects automatically
4. Resilience: Wrap calls in timeout and retry failed requests up to 2×
5. Runtime validation: Optionally decode and validate `data` against a schema
6. Error handling: Throw on `error` or `data == null`

### CRUD Wrappers
```typescript
export function getFunction<T>(name: string): Promise<T> { … }
export function postFunction<T,B = unknown>(name: string, body: B): Promise<T> { … }
export function putFunction<T,B = unknown>(name: string, body: B): Promise<T> { … }
export function patchFunction<T,B = unknown>(name: string, body: B): Promise<T> { … }
export function deleteFunction<T>(name: string): Promise<T> { … }
```

### Error Handling
```typescript
// Invoke error
if (error)
  throw new Error(`${name} invocation error: ${error.message}`)

// Missing data
if (data == null)
  throw new Error(`${name} returned no data`)
```

### Type Safety & Validation
- Annotate return types: `Promise<T>`
- Use generics for request bodies: `B = unknown`
- Implement runtime validation with schema library

### Logging (Development Only)
- Use wrapper service instead of `console.log`
- Include: function name, HTTP method, URL, response time, status, and optional payload

### Timeouts & Retries
- Wrap `invoke()` in `AbortController` with 5s timeout
- Retry network errors or 5xx responses up to two times with exponential backoff

## Examples

### Basic Function Call
```typescript
// Good
const templates = await getFunction<Template[]>('templates')

// Bad
const templates = await supabase.functions.invoke('templates')
```

### Error Handling
```typescript
// Good
try {
  const result = await postFunction('templates', { name: 'Template 1' })
} catch (error) {
  // Handle error
}

// Bad
const result = await supabase.functions.invoke('templates', {
  body: { name: 'Template 1' }
})
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agentic-k) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
