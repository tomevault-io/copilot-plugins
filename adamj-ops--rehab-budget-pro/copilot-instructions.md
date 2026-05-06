## hooks

> - `use-projects.ts` - Project queries

# Hooks & Data Fetching

## Hook Organization

### Location: `src/hooks/`

### Query Hooks (Data Fetching)
- `use-projects.ts` - Project queries
- `use-budget-items.ts` - Budget item queries
- `use-vendors.ts` - Vendor queries
- `use-dashboard.ts` - Dashboard aggregates

### Mutation Hooks (CRUD Operations)
- `use-budget-item-mutations.ts` - Budget CRUD
- `use-vendor-mutations.ts` - Vendor CRUD
- `use-draw-mutations.ts` - Draw CRUD
- `use-photo-mutations.ts` - Photo upload/delete

### Utility Hooks
- `use-auth.ts` - Authentication state
- `use-mobile.ts` - Mobile detection
- `use-sort-order.ts` - Drag & drop sorting
- `use-realtime.ts` - Supabase subscriptions
- `use-places-autocomplete.ts` - Google Places API

## Query Hook Pattern

```typescript
// src/hooks/use-projects.ts
import { useQuery } from '@tanstack/react-query';
import { getSupabaseClient } from '@/lib/supabase/client';
import type { ProjectSummary } from '@/types';

export function useProject(projectId: string) {
  return useQuery({
    queryKey: ['projects', projectId],
    queryFn: async () => {
      const supabase = getSupabaseClient();
      const { data, error } = await supabase
        .from('project_summary')  // Always use views
        .select('*')
        .eq('id', projectId)
        .single();
      if (error) throw error;
      return data as ProjectSummary;
    },
    enabled: !!projectId,
  });
}

export function useProjects() {
  return useQuery({
    queryKey: ['projects'],
    queryFn: async () => {
      const supabase = getSupabaseClient();
      const { data, error } = await supabase
        .from('project_summary')
        .select('*')
        .order('updated_at', { ascending: false });
      if (error) throw error;
      return data as ProjectSummary[];
    },
  });
}
```

## Mutation Hook Pattern

```typescript
// src/hooks/use-budget-item-mutations.ts
import { useMutation, useQueryClient } from '@tanstack/react-query';
import { getSupabaseClient } from '@/lib/supabase/client';
import { toast } from 'sonner';
import type { BudgetItemInput, BudgetItem } from '@/types';

export function useBudgetItemMutations(projectId: string) {
  const queryClient = useQueryClient();

  const createItem = useMutation({
    mutationFn: async (item: Partial<BudgetItemInput>) => {
      const supabase = getSupabaseClient();
      const { data, error } = await supabase
        .from('budget_items')
        .insert({ ...item, project_id: projectId })
        .select()
        .single();
      if (error) throw error;
      return data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['budgetItems', projectId] });
      queryClient.invalidateQueries({ queryKey: ['projects', projectId] });
      toast.success('Item added');
    },
    onError: (error) => {
      toast.error(error.message);
    },
  });

  const updateItem = useMutation({
    mutationFn: async ({ id, ...updates }: Partial<BudgetItem> & { id: string }) => {
      const supabase = getSupabaseClient();
      const { data, error } = await supabase
        .from('budget_items')
        .update(updates)
        .eq('id', id)
        .select()
        .single();
      if (error) throw error;
      return data;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['budgetItems', projectId] });
      queryClient.invalidateQueries({ queryKey: ['projects', projectId] });
    },
  });

  const deleteItem = useMutation({
    mutationFn: async (id: string) => {
      const supabase = getSupabaseClient();
      const { error } = await supabase
        .from('budget_items')
        .delete()
        .eq('id', id);
      if (error) throw error;
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['budgetItems', projectId] });
      queryClient.invalidateQueries({ queryKey: ['projects', projectId] });
      toast.success('Item deleted');
    },
  });

  const bulkDelete = useMutation({
    mutationFn: async (ids: string[]) => {
      const supabase = getSupabaseClient();
      const { error } = await supabase
        .from('budget_items')
        .delete()
        .in('id', ids);
      if (error) throw error;
    },
    onSuccess: (_, ids) => {
      queryClient.invalidateQueries({ queryKey: ['budgetItems', projectId] });
      queryClient.invalidateQueries({ queryKey: ['projects', projectId] });
      toast.success(`${ids.length} items deleted`);
    },
  });

  return { createItem, updateItem, deleteItem, bulkDelete };
}
```

## Query Key Conventions

```typescript
// Entity list
['projects']
['vendors']
['budgetItems', projectId]
['draws', projectId]

// Single entity
['projects', projectId]
['vendors', vendorId]

// Related data
['budgetCategories', projectId]
['vendorTags']
['vendorContacts', vendorId]
['lineItemPhotos', lineItemId]

// Dashboard aggregates
['portfolioSummary']
['categoryTotals']
```

## Cache Invalidation Strategy

When mutating data, invalidate related queries:

```typescript
// After budget item change
queryClient.invalidateQueries({ queryKey: ['budgetItems', projectId] });
queryClient.invalidateQueries({ queryKey: ['projects', projectId] });  // Totals change
queryClient.invalidateQueries({ queryKey: ['budgetCategories', projectId] });

// After vendor change
queryClient.invalidateQueries({ queryKey: ['vendors'] });
queryClient.invalidateQueries({ queryKey: ['vendorPaymentSummary'] });

// After draw change
queryClient.invalidateQueries({ queryKey: ['draws', projectId] });
queryClient.invalidateQueries({ queryKey: ['projects', projectId] });
```

## Optimistic Updates

For instant UI feedback:

```typescript
const updateItem = useMutation({
  mutationFn: async ({ id, ...updates }) => {
    // ... API call
  },
  onMutate: async (newData) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries({ queryKey: ['budgetItems', projectId] });
    
    // Snapshot previous value
    const previous = queryClient.getQueryData(['budgetItems', projectId]);
    
    // Optimistically update
    queryClient.setQueryData(['budgetItems', projectId], (old: BudgetItem[]) =>
      old.map(item => item.id === newData.id ? { ...item, ...newData } : item)
    );
    
    return { previous };
  },
  onError: (err, newData, context) => {
    // Rollback on error
    queryClient.setQueryData(['budgetItems', projectId], context?.previous);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ['budgetItems', projectId] });
  },
});
```

## Supabase Client Usage

```typescript
// Browser client (for client components)
import { getSupabaseClient } from '@/lib/supabase/client';
const supabase = getSupabaseClient();

// Server client (for Server Components/Route Handlers)
import { createServerClient } from '@/lib/supabase/server';
const supabase = await createServerClient();
```

---
> Source: [adamj-ops/rehab-budget-pro](https://github.com/adamj-ops/rehab-budget-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
