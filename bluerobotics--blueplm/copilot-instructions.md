## zustand

> Zustand state management architecture and slice patterns


# Zustand State Architecture

## Single Store Pattern

BluePLM uses ONE global store (`usePDMStore`) composed of slices:

```typescript
// ✅ ALWAYS import from the main store
import { usePDMStore } from '@/stores'

// ❌ NEVER create new stores
const useNewStore = create(...) // WRONG
```

## Creating a New Slice

All slices live in `src/stores/slices/`. Follow this exact pattern:

### 1. Define Types in `src/stores/types.ts`

```typescript
export interface MyFeatureSlice {
  // State
  items: Item[]
  isLoading: boolean
  
  // Actions
  setItems: (items: Item[]) => void
  addItem: (item: Item) => void
  updateItem: (id: string, updates: Partial<Item>) => void
  removeItem: (id: string) => void
  clearItems: () => void
}
```

**Naming conventions:**
- State: noun (`items`, `isLoading`, `selectedId`)
- Setters: `set` + noun (`setItems`, `setIsLoading`)
- Mutations: verb + noun (`addItem`, `removeItem`, `updateItem`)
- Bulk clear: `clear` + noun (`clearItems`)

### 2. Create Slice in `src/stores/slices/myFeatureSlice.ts`

```typescript
import { StateCreator } from 'zustand'
import type { PDMStoreState, MyFeatureSlice } from '../types'

export const createMyFeatureSlice: StateCreator<
  PDMStoreState,
  [['zustand/persist', unknown]],
  [],
  MyFeatureSlice
> = (set, get) => ({
  // Initial state
  items: [],
  isLoading: false,
  
  // Actions
  setItems: (items) => set({ items }),
  
  addItem: (item) => set((s) => ({ 
    items: [...s.items, item] 
  })),
  
  updateItem: (id, updates) => set((s) => ({
    items: s.items.map(i => i.id === id ? { ...i, ...updates } : i)
  })),
  
  removeItem: (id) => set((s) => ({
    items: s.items.filter(i => i.id !== id)
  })),
  
  clearItems: () => set({ items: [], isLoading: false })
})
```

### 3. Register in `src/stores/slices/index.ts`

```typescript
export { createMyFeatureSlice } from './myFeatureSlice'
```

### 4. Add to Combined Store in `src/stores/pdmStore.ts`

```typescript
import { createMyFeatureSlice } from './slices'

export const usePDMStore = create<PDMStoreState>()(
  persist(
    (...a) => ({
      ...createToastsSlice(...a),
      // ... existing slices ...
      ...createMyFeatureSlice(...a),  // ADD HERE
    }),
    // ... persist config
  )
)
```

### 5. Add to Combined Type in `src/stores/types.ts`

```typescript
export type PDMStoreState = 
  ToastsSlice & 
  // ... existing slices ...
  MyFeatureSlice  // ADD HERE
```

### 6. Configure Persistence (if needed)

In `pdmStore.ts`, add to `partialize`:

```typescript
partialize: (state) => ({
  // ... existing fields ...
  items: state.items,  // ADD fields to persist
})
```

## Using State in Components

### Basic Selectors

```typescript
// ✅ Select only what you need (prevents re-renders)
const items = usePDMStore(state => state.items)
const addItem = usePDMStore(state => state.addItem)

// ✅ Multiple selectors for unrelated state
const user = usePDMStore(s => s.user)
const theme = usePDMStore(s => s.theme)

// ❌ NEVER select entire store
const store = usePDMStore()  // WRONG - re-renders on ANY change
```

### Memoized Selectors for Derived State

Add complex selectors to `src/stores/selectors.ts`:

```typescript
export function useFilteredItems() {
  const items = usePDMStore(s => s.items)
  const filter = usePDMStore(s => s.filter)
  
  return useMemo(
    () => items.filter(i => i.category === filter),
    [items, filter]
  )
}
```

### Accessing State Outside React

```typescript
// For async operations, callbacks, etc.
const { items, addItem } = usePDMStore.getState()
```

## State Categories

| Scope | Use | Persist? |
|-------|-----|----------|
| User preferences | Zustand | ✅ Yes |
| UI layout | Zustand | ✅ Yes |
| Session data | Zustand | ❌ No |
| Form state | `useState` | ❌ No |
| Server cache | React Query (future) | ❌ No |

## Checklist for New Slices

- [ ] Types defined in `src/stores/types.ts`
- [ ] Slice created in `src/stores/slices/`
- [ ] Exported from `src/stores/slices/index.ts`
- [ ] Added to `PDMStoreState` type union
- [ ] Added to store composition in `pdmStore.ts`
- [ ] Persistence configured if needed
- [ ] Memoized selectors added if complex filtering needed

---
> Source: [bluerobotics/bluePLM](https://github.com/bluerobotics/bluePLM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
