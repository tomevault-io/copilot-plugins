## stillontime

> Frontend architecture patterns and React best practices


# Frontend Architecture Patterns

## 🏗️ Directory Structure
```
frontend/src/
├── components/       # React components
├── pages/           # Route-level page components
├── stores/          # Zustand state management
├── services/        # API calls and business logic
├── hooks/           # Custom React hooks
├── utils/           # Utility functions
└── types/           # TypeScript interfaces
```

## ⚛️ React Patterns

### Component Structure
```typescript
// Functional components with TypeScript
interface ComponentProps {
  title: string;
  onAction: (id: string) => void;
}

export const Component: React.FC<ComponentProps> = ({ title, onAction }) => {
  const [state, setState] = useState<string>('');
  
  return (
    <div className="component">
      <h1>{title}</h1>
      <button onClick={() => onAction('id')}>Action</button>
    </div>
  );
};
```

### Custom Hooks
```typescript
// Custom hooks for reusable logic
export const useApi = <T>(url: string) => {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const fetchData = useCallback(async () => {
    setLoading(true);
    try {
      const response = await fetch(url);
      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'Unknown error');
    } finally {
      setLoading(false);
    }
  }, [url]);

  return { data, loading, error, fetchData };
};
```

## 🗄️ State Management (Zustand)

### Store Pattern
```typescript
interface AppState {
  user: User | null;
  config: Config;
  setUser: (user: User) => void;
  setConfig: (config: Config) => void;
}

export const useAppStore = create<AppState>((set) => ({
  user: null,
  config: defaultConfig,
  setUser: (user) => set({ user }),
  setConfig: (config) => set({ config }),
}));
```

### Async Actions
```typescript
interface AsyncStore {
  data: Data[];
  loading: boolean;
  error: string | null;
  fetchData: () => Promise<void>;
}

export const useAsyncStore = create<AsyncStore>((set, get) => ({
  data: [],
  loading: false,
  error: null,
  fetchData: async () => {
    set({ loading: true, error: null });
    try {
      const response = await api.getData();
      set({ data: response, loading: false });
    } catch (error) {
      set({ error: error.message, loading: false });
    }
  },
}));
```

## 🌐 API Integration

### Service Layer
```typescript
// API service with error handling
export class ApiService {
  private baseUrl: string;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  async get<T>(endpoint: string): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`);
    if (!response.ok) {
      throw new Error(`API Error: ${response.status}`);
    }
    return response.json();
  }

  async post<T>(endpoint: string, data: any): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    if (!response.ok) {
      throw new Error(`API Error: ${response.status}`);
    }
    return response.json();
  }
}
```

### React Query Integration
```typescript
// React Query for server state
export const useUserData = (userId: string) => {
  return useQuery({
    queryKey: ['user', userId],
    queryFn: () => apiService.get<User>(`/users/${userId}`),
    enabled: !!userId,
  });
};
```

## 🎨 Styling Patterns

### Tailwind CSS
```typescript
// Utility-first styling
<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow-md">
  <h2 className="text-xl font-semibold text-gray-800">Title</h2>
  <button className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600">
    Action
  </button>
</div>
```

### Component Variants
```typescript
// Component variants with cva
const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md text-sm font-medium",
  {
    variants: {
      variant: {
        default: "bg-primary text-primary-foreground hover:bg-primary/90",
        destructive: "bg-destructive text-destructive-foreground hover:bg-destructive/90",
      },
      size: {
        default: "h-10 px-4 py-2",
        sm: "h-9 rounded-md px-3",
        lg: "h-11 rounded-md px-8",
      },
    },
    defaultVariants: {
      variant: "default",
      size: "default",
    },
  }
);
```

## 🧪 Testing Patterns

### Component Testing
```typescript
// React Testing Library
import { render, screen, fireEvent } from '@testing-library/react';
import { Component } from './Component';

describe('Component', () => {
  it('renders title correctly', () => {
    render(<Component title="Test Title" onAction={jest.fn()} />);
    expect(screen.getByText('Test Title')).toBeInTheDocument();
  });

  it('calls onAction when button is clicked', () => {
    const mockAction = jest.fn();
    render(<Component title="Test" onAction={mockAction} />);
    
    fireEvent.click(screen.getByRole('button'));
    expect(mockAction).toHaveBeenCalledWith('id');
  });
});
```

### Hook Testing
```typescript
// Testing custom hooks
import { renderHook, act } from '@testing-library/react';
import { useApi } from './useApi';

describe('useApi', () => {
  it('fetches data successfully', async () => {
    const { result } = renderHook(() => useApi('/test'));
    
    await act(async () => {
      await result.current.fetchData();
    });
    
    expect(result.current.data).toBeDefined();
    expect(result.current.loading).toBe(false);
  });
});
```

## 🚀 Performance Patterns

### Memoization
```typescript
// React.memo for component optimization
export const ExpensiveComponent = React.memo<Props>(({ data, onAction }) => {
  const processedData = useMemo(() => {
    return data.map(item => processItem(item));
  }, [data]);

  return <div>{/* render processed data */}</div>;
});
```

### Lazy Loading
```typescript
// Code splitting with React.lazy
const LazyComponent = React.lazy(() => import('./LazyComponent'));

export const App = () => (
  <Suspense fallback={<div>Loading...</div>}>
    <LazyComponent />
  </Suspense>
);
```

### Virtual Scrolling
```typescript
// For large lists
import { FixedSizeList as List } from 'react-window';

const VirtualList = ({ items }: { items: any[] }) => (
  <List
    height={600}
    itemCount={items.length}
    itemSize={50}
    itemData={items}
  >
    {({ index, style, data }) => (
      <div style={style}>
        {data[index].name}
      </div>
    )}
  </List>
);
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/makaronz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
