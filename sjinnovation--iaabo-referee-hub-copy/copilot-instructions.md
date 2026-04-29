## iaabo-referee-hub-copy

> Supabase integration patterns and best practices


# Supabase Integration Patterns

## Client Usage

- Import Supabase client from `src/integrations/supabase/client.ts`
- Use typed queries with generated types from `src/integrations/supabase/types.ts`
- Handle errors consistently with proper error messages

## Database Queries

### Reading Data
```typescript
const { data, error } = await supabase
  .from('table_name')
  .select('*')
  .eq('column', value);
```

### With React Query
- Wrap Supabase calls in React Query hooks
- Use query keys that reflect the data structure
- Handle loading and error states in components

## Authentication

- Use Supabase Auth for all authentication flows
- Access user via `useAuth` hook or context
- Protect routes that require authentication

## Row Level Security (RLS)

- All tables should have RLS enabled
- Define policies in migrations
- Test policies thoroughly

## Edge Functions

- Place functions in `supabase/functions/`
- Use TypeScript for all functions
- Handle CORS appropriately
- Validate input data

## Migrations

- Place migrations in `supabase/migrations/`
- Use descriptive migration names with timestamps
- Test migrations locally before deploying
- Document breaking changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sjinnovation) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
