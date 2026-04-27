## 09-state-management-rules

> handleSubmit,

# State Management Rules - React Context & Hooks

## Activation

- **Mode**: Always On
- **Description**: State management patterns for React Native apps

---

## State Management Hierarchy

### Choose the Right Level

```
1. Local State (useState)
   - UI state: open/closed, selected, hover
   - Form inputs
   - Temporary values

2. Lifted State (props)
   - Shared between 2-3 sibling components
   - Parent-child communication

3. Context (useContext)
   - Theme, language, auth state
   - User preferences
   - App-wide settings

4. Server State (React Query)
   - API data
   - Cached responses
   - Pagination state

5. Global State (Zustand - if needed)
   - Complex cross-feature state
   - Real-time updates
```

---

## useState Patterns

### Basic useState

```typescript
// Simple state
const [count, setCount] = useState(0);
const [name, setName] = useState('');
const [isOpen, setIsOpen] = useState(false);

// State with type annotation
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<Item[]>([]);

// Lazy initial state (for expensive computations)
const [data, setData] = useState(() => computeExpensiveInitialValue());
```

### State Update Patterns

```typescript
// CORRECT: Functional updates for state that depends on previous value
setCount((prev) => prev + 1);
setItems((prev) => [...prev, newItem]);
setUser((prev) => (prev ? { ...prev, name: newName } : null));

// WRONG: Direct reference (may cause stale state)
setCount(count + 1);
setItems([...items, newItem]);
```

### Complex State with useReducer

```typescript
// Use useReducer for complex state logic
interface FormState {
  values: Record<string, string>;
  errors: Record<string, string>;
  isSubmitting: boolean;
  isValid: boolean;
}

type FormAction =
  | { type: 'SET_FIELD'; field: string; value: string }
  | { type: 'SET_ERROR'; field: string; error: string }
  | { type: 'CLEAR_ERRORS' }
  | { type: 'SUBMIT_START' }
  | { type: 'SUBMIT_END' };

const formReducer = (state: FormState, action: FormAction): FormState => {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        values: { ...state.values, [action.field]: action.value },
        errors: { ...state.errors, [action.field]: '' },
      };
    case 'SET_ERROR':
      return {
        ...state,
        errors: { ...state.errors, [action.field]: action.error },
        isValid: false,
      };
    case 'CLEAR_ERRORS':
      return { ...state, errors: {}, isValid: true };
    case 'SUBMIT_START':
      return { ...state, isSubmitting: true };
    case 'SUBMIT_END':
      return { ...state, isSubmitting: false };
    default:
      return state;
  }
};

// Usage
const [state, dispatch] = useReducer(formReducer, initialState);
dispatch({ type: 'SET_FIELD', field: 'email', value: 'test@example.com' });
```

---

## Context Pattern

### Context Structure

```typescript
// contexts/AuthContext.tsx

// 1. Define types
interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
}

interface AuthContextValue extends AuthState {
  login: (credentials: Credentials) => Promise<void>;
  logout: () => Promise<void>;
  refreshToken: () => Promise<void>;
}

// 2. Create context with undefined default
const AuthContext = createContext<AuthContextValue | undefined>(undefined);

// 3. Create provider component
export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, setState] = useState<AuthState>({
    user: null,
    isAuthenticated: false,
    isLoading: true,
  });

  const login = useCallback(async (credentials: Credentials) => {
    setState(prev => ({ ...prev, isLoading: true }));
    try {
      const user = await authService.login(credentials);
      setState({ user, isAuthenticated: true, isLoading: false });
    } catch (error) {
      setState(prev => ({ ...prev, isLoading: false }));
      throw error;
    }
  }, []);

  const logout = useCallback(async () => {
    await authService.logout();
    setState({ user: null, isAuthenticated: false, isLoading: false });
  }, []);

  const refreshToken = useCallback(async () => {
    try {
      const user = await authService.refreshToken();
      setState(prev => ({ ...prev, user }));
    } catch {
      await logout();
    }
  }, [logout]);

  const value = useMemo(() => ({
    ...state,
    login,
    logout,
    refreshToken,
  }), [state, login, logout, refreshToken]);

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};

// 4. Create custom hook with error handling
export const useAuth = (): AuthContextValue => {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

### Context Performance Optimization

```typescript
// Split context to avoid unnecessary re-renders
// Separate state from actions

const AuthStateContext = createContext<AuthState | undefined>(undefined);
const AuthActionsContext = createContext<AuthActions | undefined>(undefined);

export const AuthProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [state, setState] = useState<AuthState>(initialState);

  // Actions are stable references
  const actions = useMemo(() => ({
    login: async (credentials: Credentials) => { /* ... */ },
    logout: async () => { /* ... */ },
  }), []);

  return (
    <AuthStateContext.Provider value={state}>
      <AuthActionsContext.Provider value={actions}>
        {children}
      </AuthActionsContext.Provider>
    </AuthStateContext.Provider>
  );
};

// Separate hooks
export const useAuthState = () => {
  const context = useContext(AuthStateContext);
  if (!context) throw new Error('useAuthState must be used within AuthProvider');
  return context;
};

export const useAuthActions = () => {
  const context = useContext(AuthActionsContext);
  if (!context) throw new Error('useAuthActions must be used within AuthProvider');
  return context;
};
```

---

## Server State with React Query

### Query Setup

```typescript
// lib/queryClient.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes
      gcTime: 10 * 60 * 1000, // 10 minutes (formerly cacheTime)
      retry: 2,
      refetchOnWindowFocus: false,
    },
  },
});
```

### Query Patterns

```typescript
// hooks/useCourses.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Query keys as constants
export const courseKeys = {
  all: ['courses'] as const,
  lists: () => [...courseKeys.all, 'list'] as const,
  list: (filters: CourseFilters) => [...courseKeys.lists(), filters] as const,
  details: () => [...courseKeys.all, 'detail'] as const,
  detail: (id: string) => [...courseKeys.details(), id] as const,
};

// Fetch hook
export const useCourses = (filters: CourseFilters) => {
  return useQuery({
    queryKey: courseKeys.list(filters),
    queryFn: () => api.getCourses(filters),
    select: (data) => data.courses, // Transform data
  });
};

// Single item hook
export const useCourse = (id: string) => {
  return useQuery({
    queryKey: courseKeys.detail(id),
    queryFn: () => api.getCourse(id),
    enabled: !!id, // Only fetch when id exists
  });
};

// Mutation hook
export const useUpdateCourse = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (data: UpdateCourseData) => api.updateCourse(data),
    onSuccess: (data, variables) => {
      // Update cache
      queryClient.setQueryData(courseKeys.detail(variables.id), data);
      // Invalidate list
      queryClient.invalidateQueries({ queryKey: courseKeys.lists() });
    },
  });
};
```

---

## Custom Hooks

### Hook Structure

```typescript
// hooks/useForm.ts
interface UseFormOptions<T> {
  initialValues: T;
  validate?: (values: T) => Partial<Record<keyof T, string>>;
  onSubmit: (values: T) => Promise<void>;
}

export const useForm = <T extends Record<string, unknown>>({
  initialValues,
  validate,
  onSubmit,
}: UseFormOptions<T>) => {
  const [values, setValues] = useState<T>(initialValues);
  const [errors, setErrors] = useState<Partial<Record<keyof T, string>>>({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const setValue = useCallback(<K extends keyof T>(field: K, value: T[K]) => {
    setValues((prev) => ({ ...prev, [field]: value }));
    setErrors((prev) => ({ ...prev, [field]: undefined }));
  }, []);

  const handleSubmit = useCallback(async () => {
    if (validate) {
      const validationErrors = validate(values);
      if (Object.keys(validationErrors).length > 0) {
        setErrors(validationErrors);
        return;
      }
    }

    setIsSubmitting(true);
    try {
      await onSubmit(values);
    } finally {
      setIsSubmitting(false);
    }
  }, [values, validate, onSubmit]);

  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
  }, [initialValues]);

  return {
    values,
    errors,
    isSubmitting,
    setValue,
    handleSubmit,
    reset,
  };
};
```

---

## State Persistence

### AsyncStorage Pattern

```typescript
// hooks/usePersistedState.ts
import AsyncStorage from '@react-native-async-storage/async-storage';

export const usePersistedState = <T>(
  key: string,
  initialValue: T,
): [T, (value: T) => void, boolean] => {
  const [value, setValue] = useState<T>(initialValue);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    AsyncStorage.getItem(key)
      .then((stored) => {
        if (stored) {
          setValue(JSON.parse(stored));
        }
      })
      .finally(() => setIsLoading(false));
  }, [key]);

  const setPersistedValue = useCallback(
    (newValue: T) => {
      setValue(newValue);
      AsyncStorage.setItem(key, JSON.stringify(newValue));
    },
    [key],
  );

  return [value, setPersistedValue, isLoading];
};
```

---

## Forbidden State Management Practices

1. **NEVER** store derived state (calculate from source state)
2. **NEVER** duplicate state across contexts
3. **NEVER** use global state for local UI state
4. **NEVER** forget cleanup in useEffect
5. **NEVER** mutate state directly
6. **NEVER** create context without custom hook
7. **NEVER** use useState for server data (use React Query)
8. **NEVER** pass unstable references to context value

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
