## 93-state-management

> Rules for state management (Zustand, Redux, React Context)


# 🔄 State Management Rules

> Auto-activated for store and state files.

## Store Structure (Zustand)

```typescript
// stores/useAuthStore.ts
import { create } from 'zustand';
import { devtools, persist } from 'zustand/middleware';

interface AuthState {
  // State
  user: User | null;
  isLoading: boolean;
  error: string | null;
  
  // Actions
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
  clearError: () => void;
}

export const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      (set, get) => ({
        // Initial state
        user: null,
        isLoading: false,
        error: null,
        
        // Actions
        login: async (credentials) => {
          set({ isLoading: true, error: null });
          try {
            const user = await authApi.login(credentials);
            set({ user, isLoading: false });
          } catch (error) {
            set({ error: error.message, isLoading: false });
          }
        },
        
        logout: () => {
          set({ user: null });
        },
        
        clearError: () => set({ error: null }),
      }),
      { name: 'auth-storage' }
    ),
    { name: 'AuthStore' }
  )
);
```

## Rules

### 1. Store Separation
```
stores/
├── useAuthStore.ts      # Authentication
├── useUIStore.ts        # UI state (modals, toasts)
├── useCartStore.ts      # Shopping cart
└── index.ts             # Re-exports
```

### 2. State Shape (STRICT)
```typescript
// ✅ GOOD: Normalized state
interface ProductsState {
  byId: Record<string, Product>;
  allIds: string[];
  isLoading: boolean;
}

// ❌ BAD: Nested arrays
interface ProductsState {
  products: Product[];          // Hard to update individual items
  categories: Category[];       // Duplicated data
}
```

### 3. Selector Pattern
```typescript
// ✅ Create selectors for computed state
export const useAuthStore = create<AuthState>()(...);

// Selectors (outside store)
export const selectUser = (state: AuthState) => state.user;
export const selectIsAuthenticated = (state: AuthState) => !!state.user;
export const selectUserRole = (state: AuthState) => state.user?.role;

// Usage
const user = useAuthStore(selectUser);
const isAuthenticated = useAuthStore(selectIsAuthenticated);
```

### 4. Action Naming
```typescript
// ✅ GOOD: Verb + noun
setUser, clearCart, addItem, removeItem, updateQuantity

// ❌ BAD: Vague names
update, change, set, do
```

## Forbidden Patterns

```typescript
// ❌ NEVER: Store sensitive data in persisted state
persist(
  (set) => ({
    accessToken: '', // ❌ Don't persist tokens!
    password: '',    // ❌ Never store passwords!
  }),
  { name: 'auth' }
)

// ❌ NEVER: Direct state mutation
const addItem = (item: Item) => {
  get().items.push(item); // Mutates directly!
  set({ items: get().items });
};

// ✅ CORRECT: Immutable update
const addItem = (item: Item) => {
  set((state) => ({ items: [...state.items, item] }));
};

// ❌ NEVER: Async logic without loading/error states
const fetchData = async () => {
  const data = await api.getData();
  set({ data }); // Missing loading/error handling!
};
```

## Testing Stores

```typescript
// stores/__tests__/useAuthStore.test.ts
import { act, renderHook } from '@testing-library/react';
import { useAuthStore } from '../useAuthStore';

describe('useAuthStore', () => {
  beforeEach(() => {
    // Reset store between tests
    useAuthStore.setState({ user: null, isLoading: false, error: null });
  });

  it('should login user', async () => {
    const { result } = renderHook(() => useAuthStore());
    
    await act(async () => {
      await result.current.login({ email: 'test@example.com', password: 'password' });
    });
    
    expect(result.current.user).toBeDefined();
    expect(result.current.isLoading).toBe(false);
  });
});
```

---
> Source: [zoxknez/ai-coding-rules](https://github.com/zoxknez/ai-coding-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
