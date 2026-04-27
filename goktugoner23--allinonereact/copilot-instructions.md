## allinonereact

> typescript-guideline

# 📘 TypeScript Best Practices Guidelines

This document outlines TypeScript best practices for the AllInOne React Native project, focusing on type safety, maintainability, and developer experience.

## 🎯 Core TypeScript Principles

### 1. Type Safety First
Leverage TypeScript's type system to catch errors at compile time, not runtime.

### 2. Explicit Over Implicit
Be explicit about types when it improves code clarity and maintainability.

### 3. Progressive Enhancement
Start with basic types and gradually add more sophisticated typing as needed.

## 🔧 Configuration Best Practices

### tsconfig.json Setup
```json
{
  "compilerOptions": {
    "target": "es2020",
    "lib": ["es2020", "dom"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-native"
  },
  "include": [
    "src/**/*",
    "types/**/*"
  ],
  "exclude": [
    "node_modules",
    "android",
    "ios"
  ]
}
```

## 🏗️ Interface & Type Definitions

### Interface Best Practices
```typescript
// Use PascalCase for interfaces
interface UserProfile {
  readonly id: string;
  name: string;
  email: string;
  avatar?: string; // Optional properties with ?
  createdAt: Date;
  settings: UserSettings;
}

// Extend interfaces for related types
interface AdminProfile extends UserProfile {
  permissions: Permission[];
  role: AdminRole;
}

// Use generic interfaces for reusability
interface ApiResponse<T> {
  data: T;
  success: boolean;
  message?: string;
  errors?: string[];
}
```

### Type Aliases vs Interfaces
```typescript
// Use type aliases for unions, primitives, and computed types
type Status = 'loading' | 'success' | 'error';
type ID = string | number;
type EventHandler<T> = (event: T) => void;

// Use interfaces for object shapes that might be extended
interface ComponentProps {
  title: string;
  onPress: () => void;
}

// Prefer interfaces for React component props
interface ButtonProps {
  title: string;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onPress: () => void;
}
```

## 🎨 React Native Specific Types

### Component Props Typing
```typescript
import { ReactNode } from 'react';
import { ViewStyle, TextStyle } from 'react-native';

interface CustomButtonProps {
  title: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  loading?: boolean;
  style?: ViewStyle;
  textStyle?: TextStyle;
  children?: ReactNode;
}

const CustomButton: React.FC<CustomButtonProps> = ({
  title,
  onPress,
  variant = 'primary',
  disabled = false,
  loading = false,
  style,
  textStyle,
  children
}) => {
  // Component implementation
};
```

### Navigation Types
```typescript
// Define navigation param lists
type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Settings: undefined;
  TransactionDetail: { transactionId: string };
};

type BottomTabParamList = {
  Home: undefined;
  Investments: undefined;
  Reports: undefined;
};

// Use with navigation hooks
import { NavigationProp, RouteProp } from '@react-navigation/native';

type ProfileScreenNavigationProp = NavigationProp<RootStackParamList, 'Profile'>;
type ProfileScreenRouteProp = RouteProp<RootStackParamList, 'Profile'>;

interface ProfileScreenProps {
  navigation: ProfileScreenNavigationProp;
  route: ProfileScreenRouteProp;
}
```

### Redux/State Types
```typescript
// Define state interfaces
interface AuthState {
  user: User | null;
  isLoading: boolean;
  error: string | null;
}

interface TransactionState {
  transactions: Transaction[];
  isLoading: boolean;
  filter: TransactionFilter;
}

// Root state type
interface RootState {
  auth: AuthState;
  transactions: TransactionState;
  investments: InvestmentState;
}

// Action types
interface LoginAction {
  type: 'auth/login';
  payload: { email: string; password: string };
}

interface LoginSuccessAction {
  type: 'auth/loginSuccess';
  payload: User;
}

type AuthAction = LoginAction | LoginSuccessAction;
```

## 🔥 Advanced TypeScript Patterns

### Utility Types Usage
```typescript
// Pick - Select specific properties
type UserPreview = Pick<User, 'id' | 'name' | 'avatar'>;

// Omit - Exclude specific properties
type CreateUserRequest = Omit<User, 'id' | 'createdAt'>;

// Partial - Make all properties optional
type UpdateUserRequest = Partial<Pick<User, 'name' | 'email' | 'avatar'>>;

// Required - Make all properties required
type CompleteUserProfile = Required<UserProfile>;

// Record - Create object type with specific keys
type TransactionsByCategory = Record<TransactionCategory, Transaction[]>;
```

### Conditional Types
```typescript
// Create conditional types for API responses
type ApiResult<T> = T extends string 
  ? { message: T }
  : { data: T };

// Conditional props based on variant
type ButtonProps<T extends 'primary' | 'secondary'> = {
  variant: T;
  title: string;
} & (T extends 'primary' 
  ? { color?: never } 
  : { color: string });
```

### Generic Constraints
```typescript
// Constrain generics for better type safety
interface Repository<T extends { id: string }> {
  findById(id: string): Promise<T | null>;
  create(item: Omit<T, 'id'>): Promise<T>;
  update(id: string, updates: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

// Use with specific types
class TransactionRepository implements Repository<Transaction> {
  async findById(id: string): Promise<Transaction | null> {
    // Implementation
  }
  // ... other methods
}
```

### Mapped Types
```typescript
// Create readonly versions of types
type ReadonlyTransaction = {
  readonly [K in keyof Transaction]: Transaction[K];
};

// Create optional versions
type PartialUpdate<T> = {
  [K in keyof T]?: T[K];
};

// Transform types
type Stringify<T> = {
  [K in keyof T]: string;
};
```

## 🔍 Type Guards & Assertions

### Type Guards
```typescript
// User-defined type guards
const isString = (value: unknown): value is string => {
  return typeof value === 'string';
};

const isUser = (value: unknown): value is User => {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'email' in value &&
    typeof (value as any).id === 'string'
  );
};

// Discriminated unions
type ApiResponse<T> = 
  | { success: true; data: T }
  | { success: false; error: string };

const handleResponse = <T>(response: ApiResponse<T>) => {
  if (response.success) {
    // TypeScript knows this is success case
    console.log(response.data);
  } else {
    // TypeScript knows this is error case
    console.error(response.error);
  }
};
```

### Type Assertions (Use Sparingly)
```typescript
// When you know more than TypeScript
const userInput = getUserInput() as string;

// Better: Use type guards instead
if (isString(userInput)) {
  // userInput is now typed as string
  console.log(userInput.toUpperCase());
}

// Non-null assertion (use carefully)
const element = document.getElementById('myElement')!;
```

## 🎪 Error Handling Types

### Result Type Pattern
```typescript
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

const fetchUser = async (id: string): Promise<Result<User>> => {
  try {
    const user = await api.getUser(id);
    return { success: true, data: user };
  } catch (error) {
    return { success: false, error: error as Error };
  }
};

// Usage
const result = await fetchUser('123');
if (result.success) {
  console.log(result.data.name); // Type-safe access
} else {
  console.error(result.error.message);
}
```

### Custom Error Types
```typescript
abstract class AppError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;
}

class ValidationError extends AppError {
  readonly code = 'VALIDATION_ERROR';
  readonly statusCode = 400;
  
  constructor(public field: string, message: string) {
    super(message);
  }
}

class NotFoundError extends AppError {
  readonly code = 'NOT_FOUND';
  readonly statusCode = 404;
}
```

## 🎣 Hooks Typing

### Custom Hook Types
```typescript
interface UseAsyncState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  execute: () => Promise<void>;
}

const useAsyncOperation = <T>(
  asyncFn: () => Promise<T>
): UseAsyncState<T> => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const execute = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const result = await asyncFn();
      setData(result);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  }, [asyncFn]);

  return { data, loading, error, execute };
};
```

### useReducer Typing
```typescript
interface CounterState {
  count: number;
  step: number;
}

type CounterAction =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset' }
  | { type: 'setStep'; payload: number };

const counterReducer = (
  state: CounterState, 
  action: CounterAction
): CounterState => {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'decrement':
      return { ...state, count: state.count - state.step };
    case 'reset':
      return { count: 0, step: 1 };
    case 'setStep':
      return { ...state, step: action.payload };
    default:
      return state;
  }
};
```

## 📊 API & Data Fetching Types

### API Response Types
```typescript
interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

interface ApiError {
  code: string;
  message: string;
  details?: Record<string, any>;
}

// Generic API client type
interface ApiClient {
  get<T>(url: string, params?: Record<string, any>): Promise<T>;
  post<T, D = any>(url: string, data: D): Promise<T>;
  put<T, D = any>(url: string, data: D): Promise<T>;
  delete<T>(url: string): Promise<T>;
}
```

### Fetch Wrapper Types
```typescript
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';

interface RequestConfig {
  method: HttpMethod;
  headers?: Record<string, string>;
  body?: any;
  timeout?: number;
}

const apiCall = async <T>(
  url: string, 
  config: RequestConfig
): Promise<Result<T, ApiError>> => {
  try {
    const response = await fetch(url, {
      method: config.method,
      headers: {
        'Content-Type': 'application/json',
        ...config.headers,
      },
      body: config.body ? JSON.stringify(config.body) : undefined,
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    const data = await response.json();
    return { success: true, data };
  } catch (error) {
    return { 
      success: false, 
      error: { 
        code: 'NETWORK_ERROR', 
        message: (error as Error).message 
      } 
    };
  }
};
```

## 🧪 Testing Types

### Test Utilities
```typescript
// Mock type helpers
type MockFunction<T extends (...args: any[]) => any> = jest.MockedFunction<T>;

type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// Factory functions for test data
const createMockUser = (overrides: DeepPartial<User> = {}): User => ({
  id: '1',
  name: 'Test User',
  email: 'test@example.com',
  createdAt: new Date(),
  ...overrides,
});

// Component testing types
interface RenderOptions {
  initialState?: DeepPartial<RootState>;
  mockFunctions?: Record<string, jest.Mock>;
}
```

## 🔐 Type Safety Best Practices

### Strict Type Checking
```typescript
// Enable strict mode in tsconfig.json
// Use strict function types
type EventHandler = (event: { type: string; payload?: any }) => void;

// Avoid any, use unknown instead
const processData = (data: unknown) => {
  if (typeof data === 'object' && data !== null) {
    // Safe to access object properties after type check
  }
};

// Use const assertions for literal types
const themes = ['light', 'dark'] as const;
type Theme = typeof themes[number]; // 'light' | 'dark'
```

### Branded Types
```typescript
// Create branded types for better type safety
type UserId = string & { readonly brand: unique symbol };
type Email = string & { readonly brand: unique symbol };

const createUserId = (id: string): UserId => id as UserId;
const createEmail = (email: string): Email => {
  if (!email.includes('@')) {
    throw new Error('Invalid email');
  }
  return email as Email;
};
```

## 🔄 Migration from JavaScript

### Gradual Migration Strategy
```typescript
// Start with basic types
interface Props {
  title: string;
  onPress: () => void;
}

// Add more specific types gradually
interface Props {
  title: string;
  variant: 'primary' | 'secondary';
  size: 'small' | 'medium' | 'large';
  onPress: (event: PressEvent) => void;
}

// Use @ts-ignore sparingly during migration
// @ts-ignore - TODO: Add proper types
const legacyFunction = require('./legacyModule');
```

## 🚫 Common Anti-Patterns

### ❌ Avoid These:
```typescript
// Don't use any
const data: any = response.data;

// Don't use function declarations without return types in complex cases
function complexCalculation(a, b) {
  return a * b + Math.random();
}

// Don't ignore TypeScript errors
// @ts-ignore
const result = unsafeOperation();

// Don't use object as a type
const config: object = { theme: 'dark' };
```

### ✅ Do This Instead:
```typescript
// Use proper types
const data: ApiResponse<User> = response.data;

// Add return types for complex functions
function complexCalculation(a: number, b: number): number {
  return a * b + Math.random();
}

// Fix TypeScript errors properly
const result = safeOperation();

// Use specific object types
interface Config {
  theme: 'light' | 'dark';
  language: string;
}
const config: Config = { theme: 'dark', language: 'en' };
```

## 📚 IDE Integration

### VS Code Settings
```json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.suggest.autoImports": true,
  "typescript.preferences.includePackageJsonAutoImports": "auto",
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  }
}
```

### ESLint TypeScript Rules
```json
{
  "@typescript-eslint/no-unused-vars": "error",
  "@typescript-eslint/no-explicit-any": "warn",
  "@typescript-eslint/explicit-function-return-type": "warn",
  "@typescript-eslint/prefer-const": "error",
  "@typescript-eslint/no-var-requires": "error"
}
```

---

**Remember**: TypeScript is a tool to make your code more maintainable and catch errors early. Use it to enhance developer experience, not to satisfy the compiler. # 📘 TypeScript Best Practices Guidelines

This document outlines TypeScript best practices for the AllInOne React Native project, focusing on type safety, maintainability, and developer experience.

## 🎯 Core TypeScript Principles

### 1. Type Safety First
Leverage TypeScript's type system to catch errors at compile time, not runtime.

### 2. Explicit Over Implicit
Be explicit about types when it improves code clarity and maintainability.

### 3. Progressive Enhancement
Start with basic types and gradually add more sophisticated typing as needed.

## 🔧 Configuration Best Practices

### tsconfig.json Setup
```json
{
  "compilerOptions": {
    "target": "es2020",
    "lib": ["es2020", "dom"],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-native"
  },
  "include": [
    "src/**/*",
    "types/**/*"
  ],
  "exclude": [
    "node_modules",
    "android",
    "ios"
  ]
}
```

## 🏗️ Interface & Type Definitions

### Interface Best Practices
```typescript
// Use PascalCase for interfaces
interface UserProfile {
  readonly id: string;
  name: string;
  email: string;
  avatar?: string; // Optional properties with ?
  createdAt: Date;
  settings: UserSettings;
}

// Extend interfaces for related types
interface AdminProfile extends UserProfile {
  permissions: Permission[];
  role: AdminRole;
}

// Use generic interfaces for reusability
interface ApiResponse<T> {
  data: T;
  success: boolean;
  message?: string;
  errors?: string[];
}
```

### Type Aliases vs Interfaces
```typescript
// Use type aliases for unions, primitives, and computed types
type Status = 'loading' | 'success' | 'error';
type ID = string | number;
type EventHandler<T> = (event: T) => void;

// Use interfaces for object shapes that might be extended
interface ComponentProps {
  title: string;
  onPress: () => void;
}

// Prefer interfaces for React component props
interface ButtonProps {
  title: string;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onPress: () => void;
}
```

## 🎨 React Native Specific Types

### Component Props Typing
```typescript
import { ReactNode } from 'react';
import { ViewStyle, TextStyle } from 'react-native';

interface CustomButtonProps {
  title: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  loading?: boolean;
  style?: ViewStyle;
  textStyle?: TextStyle;
  children?: ReactNode;
}

const CustomButton: React.FC<CustomButtonProps> = ({
  title,
  onPress,
  variant = 'primary',
  disabled = false,
  loading = false,
  style,
  textStyle,
  children
}) => {
  // Component implementation
};
```

### Navigation Types
```typescript
// Define navigation param lists
type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Settings: undefined;
  TransactionDetail: { transactionId: string };
};

type BottomTabParamList = {
  Home: undefined;
  Investments: undefined;
  Reports: undefined;
};

// Use with navigation hooks
import { NavigationProp, RouteProp } from '@react-navigation/native';

type ProfileScreenNavigationProp = NavigationProp<RootStackParamList, 'Profile'>;
type ProfileScreenRouteProp = RouteProp<RootStackParamList, 'Profile'>;

interface ProfileScreenProps {
  navigation: ProfileScreenNavigationProp;
  route: ProfileScreenRouteProp;
}
```

### Redux/State Types
```typescript
// Define state interfaces
interface AuthState {
  user: User | null;
  isLoading: boolean;
  error: string | null;
}

interface TransactionState {
  transactions: Transaction[];
  isLoading: boolean;
  filter: TransactionFilter;
}

// Root state type
interface RootState {
  auth: AuthState;
  transactions: TransactionState;
  investments: InvestmentState;
}

// Action types
interface LoginAction {
  type: 'auth/login';
  payload: { email: string; password: string };
}

interface LoginSuccessAction {
  type: 'auth/loginSuccess';
  payload: User;
}

type AuthAction = LoginAction | LoginSuccessAction;
```

## 🔥 Advanced TypeScript Patterns

### Utility Types Usage
```typescript
// Pick - Select specific properties
type UserPreview = Pick<User, 'id' | 'name' | 'avatar'>;

// Omit - Exclude specific properties
type CreateUserRequest = Omit<User, 'id' | 'createdAt'>;

// Partial - Make all properties optional
type UpdateUserRequest = Partial<Pick<User, 'name' | 'email' | 'avatar'>>;

// Required - Make all properties required
type CompleteUserProfile = Required<UserProfile>;

// Record - Create object type with specific keys
type TransactionsByCategory = Record<TransactionCategory, Transaction[]>;
```

### Conditional Types
```typescript
// Create conditional types for API responses
type ApiResult<T> = T extends string 
  ? { message: T }
  : { data: T };

// Conditional props based on variant
type ButtonProps<T extends 'primary' | 'secondary'> = {
  variant: T;
  title: string;
} & (T extends 'primary' 
  ? { color?: never } 
  : { color: string });
```

### Generic Constraints
```typescript
// Constrain generics for better type safety
interface Repository<T extends { id: string }> {
  findById(id: string): Promise<T | null>;
  create(item: Omit<T, 'id'>): Promise<T>;
  update(id: string, updates: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

// Use with specific types
class TransactionRepository implements Repository<Transaction> {
  async findById(id: string): Promise<Transaction | null> {
    // Implementation
  }
  // ... other methods
}
```

### Mapped Types
```typescript
// Create readonly versions of types
type ReadonlyTransaction = {
  readonly [K in keyof Transaction]: Transaction[K];
};

// Create optional versions
type PartialUpdate<T> = {
  [K in keyof T]?: T[K];
};

// Transform types
type Stringify<T> = {
  [K in keyof T]: string;
};
```

## 🔍 Type Guards & Assertions

### Type Guards
```typescript
// User-defined type guards
const isString = (value: unknown): value is string => {
  return typeof value === 'string';
};

const isUser = (value: unknown): value is User => {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'email' in value &&
    typeof (value as any).id === 'string'
  );
};

// Discriminated unions
type ApiResponse<T> = 
  | { success: true; data: T }
  | { success: false; error: string };

const handleResponse = <T>(response: ApiResponse<T>) => {
  if (response.success) {
    // TypeScript knows this is success case
    console.log(response.data);
  } else {
    // TypeScript knows this is error case
    console.error(response.error);
  }
};
```

### Type Assertions (Use Sparingly)
```typescript
// When you know more than TypeScript
const userInput = getUserInput() as string;

// Better: Use type guards instead
if (isString(userInput)) {
  // userInput is now typed as string
  console.log(userInput.toUpperCase());
}

// Non-null assertion (use carefully)
const element = document.getElementById('myElement')!;
```

## 🎪 Error Handling Types

### Result Type Pattern
```typescript
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };

const fetchUser = async (id: string): Promise<Result<User>> => {
  try {
    const user = await api.getUser(id);
    return { success: true, data: user };
  } catch (error) {
    return { success: false, error: error as Error };
  }
};

// Usage
const result = await fetchUser('123');
if (result.success) {
  console.log(result.data.name); // Type-safe access
} else {
  console.error(result.error.message);
}
```

### Custom Error Types
```typescript
abstract class AppError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;
}

class ValidationError extends AppError {
  readonly code = 'VALIDATION_ERROR';
  readonly statusCode = 400;
  
  constructor(public field: string, message: string) {
    super(message);
  }
}

class NotFoundError extends AppError {
  readonly code = 'NOT_FOUND';
  readonly statusCode = 404;
}
```

## 🎣 Hooks Typing

### Custom Hook Types
```typescript
interface UseAsyncState<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  execute: () => Promise<void>;
}

const useAsyncOperation = <T>(
  asyncFn: () => Promise<T>
): UseAsyncState<T> => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<Error | null>(null);

  const execute = useCallback(async () => {
    try {
      setLoading(true);
      setError(null);
      const result = await asyncFn();
      setData(result);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  }, [asyncFn]);

  return { data, loading, error, execute };
};
```

### useReducer Typing
```typescript
interface CounterState {
  count: number;
  step: number;
}

type CounterAction =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset' }
  | { type: 'setStep'; payload: number };

const counterReducer = (
  state: CounterState, 
  action: CounterAction
): CounterState => {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'decrement':
      return { ...state, count: state.count - state.step };
    case 'reset':
      return { count: 0, step: 1 };
    case 'setStep':
      return { ...state, step: action.payload };
    default:
      return state;
  }
};
```

## 📊 API & Data Fetching Types

### API Response Types
```typescript
interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

interface ApiError {
  code: string;
  message: string;
  details?: Record<string, any>;
}

// Generic API client type
interface ApiClient {
  get<T>(url: string, params?: Record<string, any>): Promise<T>;
  post<T, D = any>(url: string, data: D): Promise<T>;
  put<T, D = any>(url: string, data: D): Promise<T>;
  delete<T>(url: string): Promise<T>;
}
```

### Fetch Wrapper Types
```typescript
type HttpMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';

interface RequestConfig {
  method: HttpMethod;
  headers?: Record<string, string>;
  body?: any;
  timeout?: number;
}

const apiCall = async <T>(
  url: string, 
  config: RequestConfig
): Promise<Result<T, ApiError>> => {
  try {
    const response = await fetch(url, {
      method: config.method,
      headers: {
        'Content-Type': 'application/json',
        ...config.headers,
      },
      body: config.body ? JSON.stringify(config.body) : undefined,
    });

    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }

    const data = await response.json();
    return { success: true, data };
  } catch (error) {
    return { 
      success: false, 
      error: { 
        code: 'NETWORK_ERROR', 
        message: (error as Error).message 
      } 
    };
  }
};
```

## 🧪 Testing Types

### Test Utilities
```typescript
// Mock type helpers
type MockFunction<T extends (...args: any[]) => any> = jest.MockedFunction<T>;

type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// Factory functions for test data
const createMockUser = (overrides: DeepPartial<User> = {}): User => ({
  id: '1',
  name: 'Test User',
  email: 'test@example.com',
  createdAt: new Date(),
  ...overrides,
});

// Component testing types
interface RenderOptions {
  initialState?: DeepPartial<RootState>;
  mockFunctions?: Record<string, jest.Mock>;
}
```

## 🔐 Type Safety Best Practices

### Strict Type Checking
```typescript
// Enable strict mode in tsconfig.json
// Use strict function types
type EventHandler = (event: { type: string; payload?: any }) => void;

// Avoid any, use unknown instead
const processData = (data: unknown) => {
  if (typeof data === 'object' && data !== null) {
    // Safe to access object properties after type check
  }
};

// Use const assertions for literal types
const themes = ['light', 'dark'] as const;
type Theme = typeof themes[number]; // 'light' | 'dark'
```

### Branded Types
```typescript
// Create branded types for better type safety
type UserId = string & { readonly brand: unique symbol };
type Email = string & { readonly brand: unique symbol };

const createUserId = (id: string): UserId => id as UserId;
const createEmail = (email: string): Email => {
  if (!email.includes('@')) {
    throw new Error('Invalid email');
  }
  return email as Email;
};
```

## 🔄 Migration from JavaScript

### Gradual Migration Strategy
```typescript
// Start with basic types
interface Props {
  title: string;
  onPress: () => void;
}

// Add more specific types gradually
interface Props {
  title: string;
  variant: 'primary' | 'secondary';
  size: 'small' | 'medium' | 'large';
  onPress: (event: PressEvent) => void;
}

// Use @ts-ignore sparingly during migration
// @ts-ignore - TODO: Add proper types
const legacyFunction = require('./legacyModule');
```

## 🚫 Common Anti-Patterns

### ❌ Avoid These:
```typescript
// Don't use any
const data: any = response.data;

// Don't use function declarations without return types in complex cases
function complexCalculation(a, b) {
  return a * b + Math.random();
}

// Don't ignore TypeScript errors
// @ts-ignore
const result = unsafeOperation();

// Don't use object as a type
const config: object = { theme: 'dark' };
```

### ✅ Do This Instead:
```typescript
// Use proper types
const data: ApiResponse<User> = response.data;

// Add return types for complex functions
function complexCalculation(a: number, b: number): number {
  return a * b + Math.random();
}

// Fix TypeScript errors properly
const result = safeOperation();

// Use specific object types
interface Config {
  theme: 'light' | 'dark';
  language: string;
}
const config: Config = { theme: 'dark', language: 'en' };
```

## 📚 IDE Integration

### VS Code Settings
```json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "typescript.suggest.autoImports": true,
  "typescript.preferences.includePackageJsonAutoImports": "auto",
  "editor.codeActionsOnSave": {
    "source.organizeImports": true
  }
}
```

### ESLint TypeScript Rules
```json
{
  "@typescript-eslint/no-unused-vars": "error",
  "@typescript-eslint/no-explicit-any": "warn",
  "@typescript-eslint/explicit-function-return-type": "warn",
  "@typescript-eslint/prefer-const": "error",
  "@typescript-eslint/no-var-requires": "error"
}
```

---

**Remember**: TypeScript is a tool to make your code more maintainable and catch errors early. Use it to enhance developer experience, not to satisfy the compiler. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/goktugoner23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
