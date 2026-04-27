## 12-error-handling-rules

> - **Mode**: Always On

# Error Handling Rules - React Native

## Activation

- **Mode**: Always On
- **Description**: Error handling patterns for robust React Native apps

---

## Error Boundary

### Global Error Boundary

```typescript
// components/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';
import { View, StyleSheet } from 'react-native';
import { Text, Button } from '@/components/ui';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log to error reporting service
    console.error('ErrorBoundary caught:', error, errorInfo);

    // Call optional error handler
    this.props.onError?.(error, errorInfo);

    // Report to analytics (Sentry, etc.)
    // errorReporting.captureException(error, { extra: errorInfo });
  }

  handleRetry = () => {
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }

      return (
        <View style={styles.container}>
          <Text variant="h2" style={styles.title}>
            Något gick fel
          </Text>
          <Text variant="body" style={styles.message}>
            Vi kunde inte ladda innehållet. Försök igen.
          </Text>
          <Button onPress={this.handleRetry} label="Försök igen" />
        </View>
      );
    }

    return this.props.children;
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 24,
    backgroundColor: '#0C0A17',
  },
  title: {
    marginBottom: 12,
    color: '#F9FAFB',
  },
  message: {
    marginBottom: 24,
    textAlign: 'center',
    color: '#9CA3AF',
  },
});
```

### Using Error Boundaries

```typescript
// Wrap screens or feature sections
const App = () => (
  <ErrorBoundary
    onError={(error) => {
      // Report to monitoring service
      reportError(error);
    }}
  >
    <NavigationContainer>
      <AppNavigator />
    </NavigationContainer>
  </ErrorBoundary>
);

// Feature-level boundaries
const CourseScreen = () => (
  <ErrorBoundary fallback={<CourseErrorFallback />}>
    <CourseContent />
  </ErrorBoundary>
);
```

---

## Async Error Handling

### API Error Handling Pattern

```typescript
// types/errors.ts
export class ApiError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public code?: string,
    public details?: unknown,
  ) {
    super(message);
    this.name = 'ApiError';
  }

  static fromResponse(response: Response, body: unknown): ApiError {
    const message = (body as { message?: string })?.message || 'Unknown error';
    const code = (body as { code?: string })?.code;
    return new ApiError(message, response.status, code, body);
  }

  get isUnauthorized(): boolean {
    return this.statusCode === 401;
  }

  get isNotFound(): boolean {
    return this.statusCode === 404;
  }

  get isServerError(): boolean {
    return this.statusCode >= 500;
  }
}

export class NetworkError extends Error {
  constructor(message: string = 'Network error') {
    super(message);
    this.name = 'NetworkError';
  }
}

export class ValidationError extends Error {
  constructor(
    message: string,
    public fields: Record<string, string[]>,
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}
```

### API Client with Error Handling

```typescript
// lib/api.ts
const apiClient = async <T>(endpoint: string, options?: RequestInit): Promise<T> => {
  try {
    const response = await fetch(`${API_BASE_URL}${endpoint}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        ...options?.headers,
      },
    });

    const body = await response.json().catch(() => ({}));

    if (!response.ok) {
      throw ApiError.fromResponse(response, body);
    }

    return body as T;
  } catch (error) {
    if (error instanceof ApiError) {
      throw error;
    }

    if (error instanceof TypeError && error.message.includes('Network')) {
      throw new NetworkError('Unable to connect to server');
    }

    throw new ApiError('An unexpected error occurred', 500);
  }
};
```

---

## React Query Error Handling

### Query Error Handling

```typescript
// hooks/useCourse.ts
import { useQuery } from '@tanstack/react-query';
import { ApiError } from '@/types/errors';

export const useCourse = (courseId: string) => {
  return useQuery({
    queryKey: ['course', courseId],
    queryFn: () => api.getCourse(courseId),
    retry: (failureCount, error) => {
      // Don't retry on 404
      if (error instanceof ApiError && error.isNotFound) {
        return false;
      }
      // Retry up to 3 times for other errors
      return failureCount < 3;
    },
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
  });
};

// Usage in component
const CourseScreen = ({ courseId }: Props) => {
  const { data, error, isLoading, isError, refetch } = useCourse(courseId);

  if (isLoading) {
    return <LoadingState />;
  }

  if (isError) {
    if (error instanceof ApiError && error.isNotFound) {
      return <NotFoundState message="Kursen hittades inte" />;
    }

    return (
      <ErrorState
        message="Kunde inte ladda kursen"
        onRetry={refetch}
      />
    );
  }

  return <CourseContent course={data} />;
};
```

### Mutation Error Handling

```typescript
// hooks/useUpdateProfile.ts
export const useUpdateProfile = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: api.updateProfile,
    onError: (error) => {
      if (error instanceof ValidationError) {
        // Handle validation errors (show in form)
        return;
      }

      if (error instanceof ApiError && error.isUnauthorized) {
        // Handle auth error (logout, redirect)
        return;
      }

      // Show generic error toast
      showToast({ type: 'error', message: 'Något gick fel' });
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['profile'] });
      showToast({ type: 'success', message: 'Profilen uppdaterad' });
    },
  });
};
```

---

## Form Validation Errors

### Validation Error Display

```typescript
// components/FormField.tsx
interface FormFieldProps {
  label: string;
  value: string;
  onChangeText: (text: string) => void;
  error?: string;
  touched?: boolean;
}

export const FormField: React.FC<FormFieldProps> = ({
  label,
  value,
  onChangeText,
  error,
  touched,
}) => {
  const showError = touched && error;

  return (
    <View style={styles.container}>
      <Text style={styles.label}>{label}</Text>
      <TextInput
        value={value}
        onChangeText={onChangeText}
        style={[
          styles.input,
          showError && styles.inputError,
        ]}
        accessibilityLabel={label}
        accessibilityState={{ invalid: !!showError }}
      />
      {showError && (
        <Text style={styles.errorText} accessibilityRole="alert">
          {error}
        </Text>
      )}
    </View>
  );
};
```

### Validation Schema

```typescript
// lib/validation.ts
import { z } from 'zod';

export const loginSchema = z.object({
  email: z.string().min(1, 'E-post krävs').email('Ogiltig e-postadress'),
  password: z.string().min(1, 'Lösenord krävs').min(8, 'Lösenordet måste vara minst 8 tecken'),
});

export const validateForm = <T extends z.ZodSchema>(
  schema: T,
  data: unknown,
): { success: true; data: z.infer<T> } | { success: false; errors: Record<string, string> } => {
  const result = schema.safeParse(data);

  if (result.success) {
    return { success: true, data: result.data };
  }

  const errors: Record<string, string> = {};
  result.error.errors.forEach((err) => {
    const path = err.path.join('.');
    errors[path] = err.message;
  });

  return { success: false, errors };
};
```

---

## Toast/Snackbar Notifications

### Toast System

```typescript
// contexts/ToastContext.tsx
interface Toast {
  id: string;
  type: 'success' | 'error' | 'warning' | 'info';
  message: string;
  duration?: number;
}

interface ToastContextValue {
  toasts: Toast[];
  showToast: (toast: Omit<Toast, 'id'>) => void;
  hideToast: (id: string) => void;
}

export const ToastProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [toasts, setToasts] = useState<Toast[]>([]);

  const showToast = useCallback((toast: Omit<Toast, 'id'>) => {
    const id = Date.now().toString();
    const duration = toast.duration ?? 3000;

    setToasts(prev => [...prev, { ...toast, id }]);

    if (duration > 0) {
      setTimeout(() => {
        hideToast(id);
      }, duration);
    }
  }, []);

  const hideToast = useCallback((id: string) => {
    setToasts(prev => prev.filter(t => t.id !== id));
  }, []);

  return (
    <ToastContext.Provider value={{ toasts, showToast, hideToast }}>
      {children}
      <ToastContainer toasts={toasts} onHide={hideToast} />
    </ToastContext.Provider>
  );
};
```

---

## Offline Error Handling

### Network State Hook

```typescript
// hooks/useNetworkState.ts
import NetInfo, { NetInfoState } from '@react-native-community/netinfo';

export const useNetworkState = () => {
  const [isConnected, setIsConnected] = useState<boolean | null>(null);

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener((state: NetInfoState) => {
      setIsConnected(state.isConnected);
    });

    return () => unsubscribe();
  }, []);

  return { isConnected, isOffline: isConnected === false };
};

// Usage
const DataScreen = () => {
  const { isOffline } = useNetworkState();

  if (isOffline) {
    return (
      <OfflineState
        message="Ingen internetanslutning"
        description="Kontrollera din anslutning och försök igen"
      />
    );
  }

  return <DataContent />;
};
```

---

## Error Logging

### Error Reporting Setup

```typescript
// lib/errorReporting.ts
interface ErrorReport {
  error: Error;
  context?: Record<string, unknown>;
  user?: { id: string; email: string };
}

export const reportError = ({ error, context, user }: ErrorReport) => {
  // Log to console in development
  if (__DEV__) {
    console.error('Error Report:', error, context);
    return;
  }

  // Send to error reporting service (Sentry, etc.)
  // Sentry.captureException(error, {
  //   extra: context,
  //   user,
  // });
};

// Use throughout app
try {
  await riskyOperation();
} catch (error) {
  reportError({
    error: error as Error,
    context: { operation: 'riskyOperation', param: value },
  });
}
```

---

## Forbidden Error Handling Practices

1. **NEVER** swallow errors silently (empty catch blocks)
2. **NEVER** show technical error messages to users
3. **NEVER** forget to handle loading and error states
4. **NEVER** use try/catch without proper error typing
5. **NEVER** ignore network errors in async operations
6. **NEVER** skip Error Boundaries for critical sections
7. **NEVER** log sensitive data in error messages
8. **NEVER** use generic error messages without context

---
> Source: [denker-systems/aiklubben-app](https://github.com/denker-systems/aiklubben-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
