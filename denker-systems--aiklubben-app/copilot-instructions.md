## 11-testing-rules

> - **Mode**: Always On

# Testing Rules - React Native Testing

## Activation

- **Mode**: Always On
- **Description**: Testing standards for React Native applications

---

## Testing Stack

### Required Testing Libraries

```json
// package.json devDependencies
{
  "devDependencies": {
    "@testing-library/react-native": "^12.x",
    "@testing-library/jest-native": "^5.x",
    "jest": "^29.x",
    "jest-expo": "^50.x",
    "@types/jest": "^29.x"
  }
}
```

### Jest Configuration

```javascript
// jest.config.js
module.exports = {
  preset: 'jest-expo',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  transformIgnorePatterns: [
    'node_modules/(?!((jest-)?react-native|@react-native(-community)?)|expo(nent)?|@expo(nent)?/.*|@expo-google-fonts/.*|react-navigation|@react-navigation/.*|@unimodules/.*|unimodules|sentry-expo|native-base|react-native-svg|moti)',
  ],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1',
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.{ts,tsx}',
    '!src/**/index.{ts,tsx}',
  ],
};
```

### Jest Setup

```typescript
// jest.setup.ts
import '@testing-library/jest-native/extend-expect';

// Mock react-native-reanimated
jest.mock('react-native-reanimated', () => {
  const Reanimated = require('react-native-reanimated/mock');
  Reanimated.default.call = () => {};
  return Reanimated;
});

// Mock expo-haptics
jest.mock('expo-haptics', () => ({
  impactAsync: jest.fn(),
  notificationAsync: jest.fn(),
  selectionAsync: jest.fn(),
  ImpactFeedbackStyle: {
    Light: 'light',
    Medium: 'medium',
    Heavy: 'heavy',
  },
  NotificationFeedbackType: {
    Success: 'success',
    Warning: 'warning',
    Error: 'error',
  },
}));

// Mock safe area context
jest.mock('react-native-safe-area-context', () => ({
  SafeAreaProvider: ({ children }: { children: React.ReactNode }) => children,
  SafeAreaView: ({ children }: { children: React.ReactNode }) => children,
  useSafeAreaInsets: () => ({ top: 0, bottom: 0, left: 0, right: 0 }),
}));

// Silence console warnings in tests
global.console = {
  ...console,
  warn: jest.fn(),
  error: jest.fn(),
};
```

---

## Component Testing

### Test File Structure

```typescript
// ComponentName.test.tsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react-native';
import { ComponentName } from './ComponentName';

// Group tests with describe
describe('ComponentName', () => {
  // Setup before each test
  beforeEach(() => {
    jest.clearAllMocks();
  });

  describe('rendering', () => {
    it('renders correctly with required props', () => {
      render(<ComponentName title="Test" />);
      expect(screen.getByText('Test')).toBeTruthy();
    });

    it('renders loading state', () => {
      render(<ComponentName title="Test" isLoading />);
      expect(screen.getByTestId('loading-indicator')).toBeTruthy();
    });
  });

  describe('interactions', () => {
    it('calls onPress when pressed', () => {
      const onPress = jest.fn();
      render(<ComponentName title="Test" onPress={onPress} />);

      fireEvent.press(screen.getByRole('button'));

      expect(onPress).toHaveBeenCalledTimes(1);
    });
  });
});
```

### Querying Elements

```typescript
// PREFERRED: Use accessible queries
// 1. getByRole - most accessible
screen.getByRole('button', { name: 'Submit' });

// 2. getByLabelText - for accessibility labels
screen.getByLabelText('Username input');

// 3. getByText - for visible text
screen.getByText('Welcome');

// 4. getByTestId - last resort
screen.getByTestId('custom-component');

// Query variants
// get* - throws if not found
// query* - returns null if not found
// find* - async, waits for element

// Examples
const button = screen.getByRole('button'); // Throws if not found
const maybeButton = screen.queryByRole('button'); // Returns null if not found
const asyncButton = await screen.findByRole('button'); // Waits up to 1000ms
```

### Testing User Interactions

```typescript
import { fireEvent, waitFor } from '@testing-library/react-native';

// Press events
fireEvent.press(element);

// Text input
fireEvent.changeText(input, 'new value');

// Scroll events
fireEvent.scroll(scrollView, {
  nativeEvent: {
    contentOffset: { y: 500 },
  },
});

// Wait for async updates
await waitFor(() => {
  expect(screen.getByText('Loaded')).toBeTruthy();
});

// Wait for element to disappear
await waitFor(() => {
  expect(screen.queryByText('Loading')).toBeNull();
});
```

---

## Hook Testing

### Testing Custom Hooks

```typescript
import { renderHook, act } from '@testing-library/react-native';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with default value', () => {
    const { result } = renderHook(() => useCounter());
    expect(result.current.count).toBe(0);
  });

  it('initializes with provided value', () => {
    const { result } = renderHook(() => useCounter(10));
    expect(result.current.count).toBe(10);
  });

  it('increments count', () => {
    const { result } = renderHook(() => useCounter());

    act(() => {
      result.current.increment();
    });

    expect(result.current.count).toBe(1);
  });

  it('decrements count', () => {
    const { result } = renderHook(() => useCounter(5));

    act(() => {
      result.current.decrement();
    });

    expect(result.current.count).toBe(4);
  });
});
```

### Testing Hooks with Context

```typescript
import { renderHook } from '@testing-library/react-native';
import { useAuth } from './useAuth';
import { AuthProvider } from '@/contexts/AuthContext';

const wrapper = ({ children }: { children: React.ReactNode }) => (
  <AuthProvider>{children}</AuthProvider>
);

describe('useAuth', () => {
  it('provides auth context', () => {
    const { result } = renderHook(() => useAuth(), { wrapper });

    expect(result.current.isAuthenticated).toBe(false);
    expect(result.current.user).toBeNull();
  });
});
```

---

## Testing with Providers

### Test Utilities

```typescript
// test-utils.tsx
import React from 'react';
import { render, RenderOptions } from '@testing-library/react-native';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { NavigationContainer } from '@react-navigation/native';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { AuthProvider } from '@/contexts/AuthContext';

const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false },
    },
  });

interface AllProvidersProps {
  children: React.ReactNode;
}

const AllProviders = ({ children }: AllProvidersProps) => {
  const queryClient = createTestQueryClient();

  return (
    <QueryClientProvider client={queryClient}>
      <SafeAreaProvider>
        <NavigationContainer>
          <AuthProvider>
            {children}
          </AuthProvider>
        </NavigationContainer>
      </SafeAreaProvider>
    </QueryClientProvider>
  );
};

const customRender = (
  ui: React.ReactElement,
  options?: Omit<RenderOptions, 'wrapper'>
) => render(ui, { wrapper: AllProviders, ...options });

export * from '@testing-library/react-native';
export { customRender as render };
```

---

## Mocking

### Mocking Modules

```typescript
// Mock at top of file
jest.mock('@/services/api', () => ({
  fetchUser: jest.fn(),
  updateUser: jest.fn(),
}));

// Import mocked module
import { fetchUser, updateUser } from '@/services/api';

// Type assertion for mocked functions
const mockFetchUser = fetchUser as jest.MockedFunction<typeof fetchUser>;

// Use in tests
describe('UserProfile', () => {
  beforeEach(() => {
    mockFetchUser.mockResolvedValue({
      id: '1',
      name: 'Test User',
      email: 'test@example.com',
    });
  });

  it('displays user data', async () => {
    render(<UserProfile userId="1" />);

    await waitFor(() => {
      expect(screen.getByText('Test User')).toBeTruthy();
    });
  });
});
```

### Mocking Navigation

```typescript
const mockNavigate = jest.fn();
const mockGoBack = jest.fn();

jest.mock('@react-navigation/native', () => ({
  ...jest.requireActual('@react-navigation/native'),
  useNavigation: () => ({
    navigate: mockNavigate,
    goBack: mockGoBack,
  }),
  useRoute: () => ({
    params: { userId: '123' },
  }),
}));
```

---

## Snapshot Testing

### When to Use Snapshots

```typescript
// USE snapshots for:
// - Static UI components
// - Component variations
// - Error states

describe('Button', () => {
  it('matches snapshot for primary variant', () => {
    const tree = render(<Button variant="primary">Click</Button>);
    expect(tree.toJSON()).toMatchSnapshot();
  });

  it('matches snapshot for disabled state', () => {
    const tree = render(<Button disabled>Click</Button>);
    expect(tree.toJSON()).toMatchSnapshot();
  });
});

// DON'T use snapshots for:
// - Dynamic content
// - Frequently changing components
// - Large component trees
```

---

## Test Coverage Requirements

### Minimum Coverage Thresholds

```javascript
// jest.config.js
module.exports = {
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
};
```

### What to Test

```
Priority 1 (Required):
- Custom hooks
- Utility functions
- Form validation
- Critical user flows

Priority 2 (Recommended):
- UI components with logic
- Navigation flows
- Error handling

Priority 3 (Optional):
- Simple presentational components
- Styling variations
```

---

## Forbidden Testing Practices

1. **NEVER** test implementation details (internal state)
2. **NEVER** use snapshot tests for dynamic content
3. **NEVER** skip cleanup in async tests
4. **NEVER** use test timeouts without justification
5. **NEVER** test third-party library functionality
6. **NEVER** write tests that depend on test order
7. **NEVER** use real API calls in unit tests
8. **NEVER** ignore flaky tests (fix or remove them)

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
