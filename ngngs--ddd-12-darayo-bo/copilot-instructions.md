## ddd-12-darayo-bo

> API patterns and data fetching conventions


# API and Data Fetching Conventions

## API Layer Structure
All API calls go through [lib/api.ts](mdc:lib/api.ts) which provides:

### Base Configuration
- Dynamic API base URL: `localhost:8080` for development, `https://darayo-festival.shop` for production
- 30-second timeout on all requests
- Proper error handling with try/catch blocks
- JSON content-type headers by default

### API Function Patterns
```typescript
// Generic API helper
async function apiCall<T>(endpoint: string, options: RequestInit = {}): Promise<T>

// Specific resource functions
export const fetchFestivals = (): Promise<FestivalResponse[]>
export const createFestival = (festivalData: FestivalCreateRequest): Promise<void>
export const updateFestival = (id: number, festivalData: FestivalCreateRequest): Promise<void>
export const deleteFestival = (id: number, password: string): Promise<void>
```

### Password Protection
Many operations require password authentication:
- Create/Update/Delete operations include password in request body
- Use PasswordModal component for collecting passwords
- Password is included in request body, not headers

### Error Handling Pattern
```typescript
try {
  const response = await fetch(url, defaultOptions);
  if (!response.ok) {
    const errorText = await response.text();
    throw new Error(`HTTP ${response.status}: ${errorText}`);
  }
  return await response.json();
} catch (error) {
  console.error(`API Error:`, error);
  throw error;
}
```

## Component API Integration
- Use `useEffect` for data fetching on component mount
- Handle loading states with boolean flags
- Show error messages to users when API calls fail
- Use optimistic updates where appropriate

Example pattern:
```typescript
const [data, setData] = useState<Type[]>([]);
const [isLoading, setIsLoading] = useState(false);

useEffect(() => {
  const loadData = async () => {
    setIsLoading(true);
    try {
      const result = await apiFunction();
      setData(result);
    } catch (error) {
      console.error('Failed to load data:', error);
    } finally {
      setIsLoading(false);
    }
  };
  loadData();
}, []);
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ngngs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ngngs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
