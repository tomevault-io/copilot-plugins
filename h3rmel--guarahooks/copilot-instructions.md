## api-design-guidelines

> This document establishes standards for creating consistent, intuitive, and robust APIs for hooks in guarahooks.

# API Design Guidelines

This document establishes standards for creating consistent, intuitive, and robust APIs for hooks in guarahooks.

## 🎯 Fundamental Principles

### 1. Consistency

- Similar APIs should work similarly
- Uniform naming conventions
- Predictable return patterns

### 2. Simplicity

- Minimal but complete APIs
- Common cases should be simple
- Optional complexity through configurations

### 3. Predictability

- Expected behavior without surprises
- Consistent error handling
- Predictable performance

## 📝 Naming Conventions

### Hook Names

```typescript
// ✅ Correct
useToggle()
useLocalStorage()
useFetch()
useDebounceState()

// ❌ Incorrect
useToggleState()      // Redundant
useToggleBoolean()    // Too specific
toggleHook()          // Doesn't follow use* convention
```

### Parameter Names

```typescript
// ✅ Established standards
initialValue          // Initial value
defaultValue          // Default value
onToggle, onChange    // Callbacks
options               // Optional configurations
key                   // Keys for storage
delay                 // Delay times
```

### Return Names

```typescript
// ✅ Established standards
value, setValue       // Simple state
data, error, loading  // Async operations
toggle, clear, reset  // Actions
isLoading, isError    // Boolean states
```

## 🔧 API Patterns

### 1. Simple State Hooks

```typescript
// Pattern: [value, setter]
function useToggle(
  initialValue: boolean = false,
  onToggle?: (value: boolean) => void
): [boolean, () => void]

function useCounter(
  initialValue: number = 0
): [number, { increment: () => void; decrement: () => void; reset: () => void }]
```

### 2. Complex State Hooks

```typescript
// Pattern: Object with named properties
function useLocalStorage<T>(
  key: string,
  initialValue: T,
  options?: UseLocalStorageOptions<T>
): {
  value: T;
  setValue: (value: T) => void;
  removeValue: () => void;
}
```

### 3. Async Hooks

```typescript
// Pattern: data, error, loading + actions
function useFetch<T>(
  url: string,
  options?: FetchOptions
): {
  data: T | null;
  error: Error | null;
  loading: boolean;
  refetch: () => Promise<T | null>;
  abort: () => void;
}
```

### 4. Effect/Sensor Hooks

```typescript
// Pattern: Current value + optional metadata
function useWindowSize(): {
  width: number;
  height: number;
}

function useGeolocation(options?: PositionOptions): {
  position: GeolocationPosition | null;
  error: GeolocationPositionError | null;
  loading: boolean;
}
```

## ⚙️ Configurations and Options

### Options Structure

```typescript
interface UseHookOptions {
  // Behavior configurations
  enabled?: boolean;        // Hook active/inactive
  immediate?: boolean;      // Immediate execution
  
  // Performance configurations
  debounce?: number;        // Debounce timing
  throttle?: number;        // Throttle timing
  
  // Callbacks
  onSuccess?: (data: T) => void;
  onError?: (error: Error) => void;
  onChange?: (value: T) => void;
  
  // Specific configurations
  // ... other relevant options
}
```

### Sensible Default Values

```typescript
// ✅ Good standards
const defaultOptions = {
  enabled: true,           // Hook active by default
  immediate: true,         // Immediate execution
  retries: 0,             // No retry by default
  timeout: 5000,          // 5s reasonable timeout
  debounce: 300,          // 300ms default debounce
};
```

## 🎭 Return Patterns

### 1. Tuple Pattern (Simple State)

```typescript
// For simple useState-like hooks
const [value, setValue] = useToggle();
const [count, { increment, decrement }] = useCounter();
```

### 2. Object Pattern (Complex State)  

```typescript
// For hooks with multiple related values
const { data, error, loading, refetch } = useFetch('/api/users');
const { position, error, loading } = useGeolocation();
```

### 3. Value Pattern (Sensors/Computed)

```typescript
// For hooks that return only a computed value
const windowSize = useWindowSize();
const isOnline = useOnlineStatus();
const deviceOrientation = useOrientation();
```

## 🔄 State Management

### Initialization

```typescript
// ✅ Correct: useState with stable initial value
const [value, setValue] = useState(() => {
  // Expensive computation only on initialization
  return computeInitialValue();
});

// ✅ Correct: useRef for stable values
const initialRef = useRef(initialValue);
```

### Updates and Callbacks

```typescript
// ✅ Correct: useCallback for stability
const handleChange = useCallback((newValue: T) => {
  setValue(newValue);
  onToggle?.(newValue);
}, [onToggle]);

// ✅ Correct: Stable update function
const toggle = useCallback(() => {
  setValue(prev => !prev);
}, []);
```

## 🚨 Error Handling

### Error Patterns

```typescript
// ✅ Consistent pattern for async hooks
interface AsyncHookReturn<T> {
  data: T | null;
  error: Error | null;
  loading: boolean;
}

// ✅ Try-catch with graceful fallback
const readValue = useCallback((): T => {
  try {
    return JSON.parse(localStorage.getItem(key)) ?? initialValue;
  } catch (error) {
    console.warn(`Error reading localStorage key "${key}":`, error);
    return initialValue;
  }
}, [key, initialValue]);
```

## 📊 Performance Guidelines

### Mandatory Optimizations

```typescript
// ✅ useCallback for stable functions
const stableFunction = useCallback(() => {
  // implementation
}, [dependencies]);

// ✅ useMemo for expensive computations
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(input);
}, [input]);

// ✅ Refs for values that don't cause re-render
const intervalRef = useRef<NodeJS.Timeout>();
```

### Cleanup

```typescript
// ✅ Always clean up resources
useEffect(() => {
  const timer = setInterval(() => {
    // action
  }, 1000);
  
  return () => clearInterval(timer);
}, []);
```

## 🧪 TypeScript Guidelines

### Type Safety

```typescript
// ✅ Generics for type-safe flexibility
function useLocalStorage<T>(
  key: string, 
  initialValue: T
): [T, (value: T) => void]

// ✅ Interfaces for complex options
interface UseFetchOptions extends RequestInit {
  timeout?: number;
  retries?: number;
}
```

### JSDoc Documentation

```typescript
/**
 * Hook to control boolean state with toggle function
 * 
 * @param initialValue - Initial state value (default: false)
 * @param onToggle - Callback called when state changes
 * @returns Tuple with [value, toggle function]
 * 
 * @example
 * ```tsx
 * const [isOpen, toggle] = useToggle(false);
 * ```
 */
export function useToggle(
  initialValue: boolean = false,
  onToggle?: (value: boolean) => void
): [boolean, () => void]
```

## ✅ API Checklist

### Design

- [ ] Hook name is descriptive and follows `use*` pattern
- [ ] Parameters follow established conventions
- [ ] Return is consistent with category pattern
- [ ] Optional configurations in `options` object
- [ ] Default values are sensible

### Implementation

- [ ] Uses TypeScript with strict types
- [ ] Has complete JSDoc
- [ ] Implements appropriate error handling
- [ ] Uses `useCallback`/`useMemo` where needed
- [ ] Performs resource cleanup

### Compatibility

- [ ] Works with SSR (returns safe values)
- [ ] Works with React Strict Mode
- [ ] Doesn't break on frequent re-renders
- [ ] Adequate performance for real-world use

Following these guidelines ensures consistent, robust, and easy-to-use APIs throughout the guarahooks ecosystem.

---
> Source: [h3rmel/guarahooks](https://github.com/h3rmel/guarahooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
