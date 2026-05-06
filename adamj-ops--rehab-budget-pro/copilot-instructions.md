## testing

> When testing features, use the browser tools:

# Testing & Debugging

## Development Server

```bash
npm run dev
# Runs on http://localhost:3000
```

## Browser Testing with Cursor

When testing features, use the browser tools:

1. Navigate to `http://localhost:3000`
2. Take snapshots to verify UI state
3. Interact with elements to test functionality
4. Check console for errors

### Common Test Flows

#### Project Creation
1. Click "New Project" button
2. Enter street address (auto-fills city/state/zip)
3. Fill in ARV, purchase price
4. Submit → Should redirect to project detail

#### Budget Editing
1. Navigate to project → Budget Detail tab
2. Click edit icon on any line item
3. Modify underwriting/forecast/actual amounts
4. Click save → Should update totals

#### Vendor Management
1. Navigate to project → Vendors tab
2. Click "Add Vendor"
3. Fill in vendor details
4. Save → Should appear in vendor list

#### Photo Upload
1. Navigate to project → Budget Detail tab
2. Click camera icon on any line item
3. Drag & drop or select file
4. Should upload and show in gallery

## Console Debugging

```typescript
// In React Query hooks
const { data, error, isLoading } = useQuery({
  queryKey: ['projects'],
  queryFn: async () => {
    console.log('Fetching projects...');
    const result = await supabase.from('project_summary').select('*');
    console.log('Result:', result);
    return result.data;
  },
});
```

## Supabase Debugging

### Check Supabase Dashboard
- SQL Editor: Test queries directly
- Table Editor: View/edit data
- Logs: Check for RLS policy errors
- Storage: Verify file uploads

### Common Issues

#### RLS Policy Errors
```
Error: new row violates row-level security policy
```
Fix: Check `user_id` is being passed in inserts (auth not implemented yet)

#### Missing Data
```typescript
// Check if query returns expected data
const { data, error } = await supabase
  .from('project_summary')
  .select('*')
  .eq('id', projectId)
  .single();

console.log('Data:', data);
console.log('Error:', error);
```

## React Query DevTools

Add to layout for debugging:

```tsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

<QueryClientProvider client={queryClient}>
  {children}
  <ReactQueryDevtools initialIsOpen={false} />
</QueryClientProvider>
```

## Error Handling

### Toast Notifications
```typescript
import { toast } from 'sonner';

// Success
toast.success('Saved successfully');

// Error
toast.error('Failed to save');

// With description
toast.error('Upload failed', {
  description: 'File size exceeds 10MB limit',
});
```

### Error Boundaries
Wrap components that might fail:

```tsx
<ErrorBoundary fallback={<ErrorFallback />}>
  <BudgetDetailTab />
</ErrorBoundary>
```

## Performance Debugging

### React Query Stale Time
```typescript
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5, // 5 minutes
      gcTime: 1000 * 60 * 30, // 30 minutes
    },
  },
});
```

### Check Re-renders
```typescript
import { useEffect } from 'react';

useEffect(() => {
  console.log('Component rendered');
});
```

## Database Migrations

### Test Migrations Locally
1. Write migration in `supabase/migrations/`
2. Test in Supabase SQL Editor
3. Verify TypeScript types match

### Rollback Pattern
```sql
-- Include rollback in migration comments
-- DOWN:
-- ALTER TABLE budget_items DROP COLUMN new_column;
```

---
> Source: [adamj-ops/rehab-budget-pro](https://github.com/adamj-ops/rehab-budget-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
