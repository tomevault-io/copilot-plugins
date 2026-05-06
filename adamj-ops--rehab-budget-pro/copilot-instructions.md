## project

> Rehab Budget Pro is a fix & flip real estate project budget tracking application for managing property deals with:

# Rehab Budget Pro - Cursor Rules

## Project Overview

Rehab Budget Pro is a fix & flip real estate project budget tracking application for managing property deals with:
- **Budget tracking** across 18 categories using a three-column model (Underwriting → Forecast → Actual)
- **Vendor management** with tags, contact history, and CSV import/export
- **Draw/payment tracking** with milestone-based workflow
- **Photo uploads** for receipts and progress documentation
- **Cost reference data** for Minneapolis metro area pricing

## Tech Stack

- **Framework**: Next.js 15 (App Router) + React 19
- **Database**: Supabase (PostgreSQL with RLS)
- **Styling**: Tailwind CSS v4 + shadcn/ui (Mira theme)
- **State**: React Query (server) + Zustand (client UI)
- **Icons**: Tabler Icons (`@tabler/icons-react`)
- **Animations**: Framer Motion + tailwindcss-animate

## Critical Architecture Rules

### Data Flow Pattern
```
User Action → React Query Mutation → Supabase → Cache Invalidation → Re-render
```

### State Management Split
- **React Query**: ALL database operations, caching, server state
- **Zustand** (`src/lib/store.ts`): UI state only (modals, active tab, selections)

### Database Views for Computed Data
NEVER calculate budget totals client-side. Always use views:
- `project_summary` - Projects with calculated totals (rehab_budget, ROI, MAO)
- `budget_by_category` - Category aggregates with variances
- `vendor_payment_summary` - Vendor totals across projects

### Type System
- Import types from `@/types` (path alias → `src/`)
- Database types mirror PostgreSQL schema exactly
- Use `*Input` types for mutations (omit id, user_id, timestamps)

## File Structure Conventions

```
src/
├── app/                    # Next.js App Router pages
│   ├── page.tsx           # Dashboard (Kanban pipeline)
│   ├── projects/[id]/     # Project detail with tabs
│   └── dashboard/         # Portfolio analytics
├── components/
│   ├── dashboard/         # Portfolio health, pipeline, analytics
│   ├── project/           # Project-specific components
│   │   └── tabs/          # Deal Summary, Budget, Vendors, Draws, Cost Ref
│   ├── ui/                # shadcn/ui primitives
│   └── vendor/            # Vendor management components
├── hooks/                 # Custom React hooks
│   ├── use-*-mutations.ts # CRUD operations per entity
│   └── use-*.ts           # Query hooks
├── lib/
│   ├── supabase/          # Client (browser) and server clients
│   ├── store.ts           # Zustand store
│   └── utils.ts           # Utility functions (cn, formatters)
└── types/
    └── index.ts           # All TypeScript types and enums
```

## Code Patterns

### React Query Mutations
```typescript
const mutation = useMutation({
  mutationFn: async (data) => {
    const supabase = getSupabaseClient();
    const { error } = await supabase.from('table').insert(data);
    if (error) throw error;
  },
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['key'] });
    toast.success('Saved');
  },
});
```

### Supabase Queries
```typescript
const { data, isLoading } = useQuery({
  queryKey: ['projects', projectId],
  queryFn: async () => {
    const supabase = getSupabaseClient();
    const { data, error } = await supabase
      .from('project_summary')  // Use views for computed data
      .select('*')
      .eq('id', projectId)
      .single();
    if (error) throw error;
    return data;
  },
});
```

## UI Guidelines

### Use CSS Utility Classes from globals.css
- Status badges: `.status-badge`, `.status-active`, `.status-pending`
- Stat cards: `.stat-card`, `.stat-value`, `.stat-label`
- Tables: `.table-header`, `.table-row-hover`
- Forms: `.form-input`, `.inline-input`
- Icons: `.icon-sm`, `.icon-md`, `.icon-lg`

### Component Library
- Use shadcn/ui components from `src/components/ui/`
- Icons: `@tabler/icons-react` (NOT lucide-react for new code)
- Sheets for forms (slide from right)
- AlertDialog for confirmations

## Database Schema Key Points

### Three-Column Budget Model
- `underwriting_amount`: Pre-deal estimate (acquisition analysis)
- `forecast_amount`: Post-walkthrough/contractor bid estimate
- `actual_amount`: Real spend during construction

### Project Status Flow
`lead` → `analyzing` → `under_contract` → `in_rehab` → `listed` → `sold`
(Can be marked `dead` at any point)

### 18 Budget Categories
soft_costs, demo, structural, plumbing, hvac, electrical, insulation_drywall,
interior_paint, flooring, tile, kitchen, bathrooms, doors_windows, interior_trim,
exterior, landscaping, finishing, contingency

## Current Implementation Status

### ✅ Completed
- Three-column budget model with inline editing
- Full CRUD for budget items, vendors, draws
- Photo upload per line item
- Vendor tags, contact history, CSV import/export
- Kanban pipeline dashboard
- Deal Summary with MAO calculations

### 🚧 In Progress
- PDF exports (underwriting summary, investor packets)
- Drag & drop reordering for budget items

### ❌ Not Started
- User authentication
- Real-time updates via Supabase subscriptions
- Budget templates (save/reuse structures)

## Common Tasks

### Adding a Database Column
1. Create migration in `supabase/migrations/`
2. Add field to TypeScript type in `src/types/index.ts`
3. Run migration in Supabase SQL Editor
4. Update components

### Adding a New UI Component
1. Check if shadcn/ui has it: `npx shadcn@latest add [component]`
2. Or create in `src/components/ui/`
3. Use Radix UI primitives + Tailwind

### Creating a Mutation Hook
1. Create `src/hooks/use-[entity]-mutations.ts`
2. Export useMutation hooks for create, update, delete
3. Invalidate relevant query keys on success

---
> Source: [adamj-ops/rehab-budget-pro](https://github.com/adamj-ops/rehab-budget-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
