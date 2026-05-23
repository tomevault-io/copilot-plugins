## conventions

> - Use strict TypeScript configuration

# 📝 Coding Conventions & Best Practices

## TypeScript Guidelines

### Type Safety
- Use strict TypeScript configuration
- Define interfaces for all data structures
- Use type assertions sparingly and with proper validation
- Prefer `interface` over `type` for object shapes

### Function Declarations
```typescript
// Prefer arrow functions for components
const MyComponent: React.FC = () => {
  return <div>Content</div>;
};

// Use async/await for asynchronous operations
const fetchData = async (): Promise<DataType> => {
  const response = await api.getData();
  return response.data;
};
```

### Error Handling
```typescript
try {
  const result = await apiCall();
  // Handle success
} catch (error) {
  console.error('Operation failed:', error);
  toast({
    title: "Error",
    description: "Operation failed. Please try again.",
    variant: "destructive"
  });
}
```

## React Patterns

### Component Structure
```typescript
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

interface ComponentProps {
  // Define props interface
}

const Component: React.FC<ComponentProps> = ({ prop1, prop2 }) => {
  // State declarations
  const [state, setState] = useState<StateType>(initialValue);
  
  // Effect hooks
  useEffect(() => {
    // Side effects
  }, [dependencies]);
  
  // Event handlers
  const handleAction = () => {
    // Handler logic
  };
  
  // Render
  return (
    <div>
      {/* JSX content */}
    </div>
  );
};

export default Component;
```

### Custom Hooks
```typescript
// Custom hook naming: use + descriptive name
const useProblemData = (problemId: string) => {
  const [data, setData] = useState<Problem | null>(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Fetch logic
  }, [problemId]);
  
  return { data, loading };
};
```

## API Integration

### API Service Structure
```typescript
export const problemApi = {
  // CRUD operations
  getProblems: async (teacherId: string): Promise<Problem[]> => {
    const { data, error } = await supabase
      .from('problems')
      .select('*')
      .eq('teacher_id', teacherId);
    
    if (error) throw error;
    return data || [];
  },
  
  createProblem: async (problemData: CreateProblemData): Promise<Problem> => {
    const { data, error } = await supabase
      .from('problems')
      .insert(problemData)
      .select()
      .single();
    
    if (error) throw error;
    return data;
  }
};
```

### Database Operations
- Always use proper error handling
- Include logging for debugging
- Use transactions for related operations
- Validate data before database operations

## State Management

### Local State
```typescript
// Use descriptive state names
const [isLoading, setIsLoading] = useState(false);
const [selectedProblems, setSelectedProblems] = useState<Set<string>>(new Set());
const [formData, setFormData] = useState<FormData>(initialFormData);
```

### Form Handling
```typescript
// Use React Hook Form for complex forms
const form = useForm<FormData>({
  resolver: zodResolver(formSchema),
  defaultValues: initialValues
});

// Handle form submission
const onSubmit = async (data: FormData) => {
  try {
    await api.createItem(data);
    toast({ title: "Success", description: "Item created successfully" });
  } catch (error) {
    toast({ 
      title: "Error", 
      description: "Failed to create item",
      variant: "destructive" 
    });
  }
};
```

## File Naming Conventions

### Components
- **PascalCase**: `TeacherDashboard.tsx`, `ProblemManagement.tsx`
- **Descriptive names**: Clearly indicate component purpose
- **Suffixes**: Use appropriate suffixes (`Dashboard`, `Management`, `Form`)

### Hooks
- **camelCase with 'use' prefix**: `useAuth.tsx`, `useProblemData.tsx`
- **Descriptive names**: Indicate hook functionality

### Utilities
- **camelCase**: `api.ts`, `utils.ts`, `constants.ts`
- **Domain-specific**: Group related utilities

## Code Organization

### Import Statements
```typescript
// 1. React and external libraries
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';

// 2. Internal components (using @/ alias)
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader } from '@/components/ui/card';

// 3. Custom hooks and utilities
import { useAuth } from '@/hooks/useAuth';
import { problemApi } from '@/lib/api';

// 4. Type definitions
import type { Problem, User } from '@/types/database';
```

### Component Props
```typescript
// Define clear prop interfaces
interface ProblemCardProps {
  problem: Problem;
  onEdit?: (problem: Problem) => void;
  onDelete?: (problemId: string) => void;
  showActions?: boolean;
}

// Use destructuring with default values
const ProblemCard: React.FC<ProblemCardProps> = ({ 
  problem, 
  onEdit, 
  onDelete, 
  showActions = true 
}) => {
  // Component logic
};
```

## Performance Best Practices

### Memoization
```typescript
// Use React.memo for expensive components
const ExpensiveComponent = React.memo<Props>(({ data }) => {
  // Component logic
});

// Use useMemo for expensive calculations
const expensiveValue = useMemo(() => {
  return heavyCalculation(data);
}, [data]);

// Use useCallback for event handlers passed to children
const handleClick = useCallback(() => {
  // Handler logic
}, [dependencies]);
```

### Lazy Loading
```typescript
// Lazy load heavy components
const HeavyComponent = lazy(() => import('./HeavyComponent'));

// Use Suspense for loading states
<Suspense fallback={<LoadingSpinner />}>
  <HeavyComponent />
</Suspense>
```

## Error Boundaries & Logging

### Error Handling
```typescript
// Always handle errors gracefully
const handleApiCall = async () => {
  try {
    setLoading(true);
    const result = await api.call();
    setData(result);
  } catch (error) {
    console.error('API call failed:', error);
    setError('Failed to load data');
  } finally {
    setLoading(false);
  }
};
```

### Logging
```typescript
// Use consistent logging format
console.log('Component mounted:', componentName);
console.error('Operation failed:', { error, context });
console.warn('Deprecated API used:', apiName);
```

## Testing Considerations

### Testable Code
- Write pure functions when possible
- Separate business logic from UI components
- Use dependency injection for external services
- Keep components focused and single-purpose

### Accessibility
- Use semantic HTML elements
- Include proper ARIA labels
- Ensure keyboard navigation works
- Test with screen readers

---
> Source: [hyunwook20230402/math-learning-management](https://github.com/hyunwook20230402/math-learning-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
