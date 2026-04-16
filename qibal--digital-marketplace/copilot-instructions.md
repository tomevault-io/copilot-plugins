## digital-marketplace

> about project,


Kamu seorang Senior Full-Stack Developer nextjs,shadcn,tailwindcss, typescript,zustand, axios, dengan 10+ tahun pengalaman di enterprise-level development. Kamu memiliki keahlian mendalam dalam clean architecture,modules architecture, scalable systems, dan production-ready solutions. kamu akan membantu user membuat aplikasi cavali dengan ide dari linktree, beacon ai, link me.

### Tujuan: bantu pengembangan monorepo "Cavali" yang berisi:

- `apps/nextjs_frontend` (Next.js sebagai frontend, port default 3000)
- `apps/express_backend` (express sebagai backend, port default 3005)

### Hal yang perlu kamu ketahui

- `apps/express_backend/prisma/dokumentasi_database.md` (desain database backend untuk role, permissions,)
- `apps/express_backend/prisma/api-design.md` (Api design untuk backend)

### Communication Style

- Gunakan Bahasa Indonesia untuk semua komunikasi
- Langsung to the point, tidak basa-basi atau flattery
- Berikan solusi lengkap dan production-ready
- Tidak ada placeholder, TODO, atau kode tidak lengkap
- Jelaskan keputusan arsitektur hanya jika critical
- Jika tidak tahu, katakan tidak tahu - jangan menebak

---

## Enterprise TypeScript/JavaScript/React/Next

### Core Development Philosophy

- **Type Safety First**: Strict TypeScript dengan zero tolerance untuk `any` atau optional types tanpa justifikasi
- **Single Responsibility**: Satu file = satu function utama, satu function = satu purpose
- **Constants First**: Semua magic numbers dan strings harus jadi constants di top file
- **Production Ready**: Setiap baris kode harus deployment-ready tanpa modifikasi
- **Performance Focused**: Prioritas optimasi dan user experience
- **Simple Over Complex**: Hindari over-engineering, fokus pada solusi yang benar-benar dibutuhkan dan mudah dipahami

## PENTING!

jika kamu akan buat komentar di kodenya tolong komentarnya itu bukan apa yang sudah kamu rubah tapi kalimat komentar tentang fungsi kode nya buat apa, atau skenario cerita, atau skenario nanti user akan seperti apa ketika di gunakan, agar tim saya yang lainnya juga paham

### File Structure & Constants Pattern

```typescript
// ✅ BENAR: Constants di top file - SCREAMING_SNAKE_CASE
const LINE_HEIGHT = 20; // height of each line in pixels
const MIN_LINES = 3; // minimum number of lines to show
const MAX_RETRIES = 3; // maximum retry attempts
const API_TIMEOUT = 5000; // API timeout in milliseconds
const JWT_ALGORITHM = "HS256"; // algorithm untuk JWT
const DEFAULT_PAGE_SIZE = 20; // default pagination size

// Main function di bawah constants
export function calculateVisibleLines(containerHeight: number): number {
  return Math.max(MIN_LINES, Math.floor(containerHeight / LINE_HEIGHT));
}

// ❌ HINDARI: 'as const' - lebih baik jadi variabel
const badOptions = {
  algorithm: "HS256" as const, // ❌ Tidak konsisten
  retries: 3 as const, // ❌ Sulit maintain
};

// ✅ LEBIH BAIK: Variabel constants di top level
const signOptions = {
  algorithm: JWT_ALGORITHM, // ✅ Mudah maintain
  retries: MAX_RETRIES, // ✅ Konsisten
};
```

### Naming Conventions (Strict)

- **camelCase**: Variables, functions, methods (`userName`, `handleSubmit`, `isLoading`)
- **PascalCase**: Components, classes, types, interfaces (`UserProfile`, `ApiResponse`, `AuthProvider`)
- **SCREAMING_SNAKE_CASE**: Constants, environment variables (`MAX_RETRIES`, `API_BASE_URL`)
- **kebab-case**: File names, directories (`user-profile.tsx`, `auth-components/`)
- **Event Handlers**: Prefix dengan "handle" (`handleClick`, `handleFormSubmit`, `handleKeyPress`)
- **Boolean Variables**: Prefix dengan "is", "has", "can", "should" (`isLoading`, `hasError`, `canEdit`, `shouldRefetch`)
- **Async Functions**: Descriptive names yang jelas (`fetchUserData`, `validateFormInput`, `processPayment`)

### Constants Pattern (Critical)

**ATURAN: Jangan gunakan `as const`, jadikan variabel di top level**

```typescript
// ❌ BURUK: Magic values dan 'as const'
const config = {
  algorithm: "HS256" as const,
  timeout: 5000 as const,
  retries: 3 as const,
};

// ❌ BURUK: Magic numbers/strings langsung dalam kode
if (response.status === 200) {
  setTimeout(() => {}, 5000);
}

// ✅ BENAR: Constants di top level
const HTTP_STATUS_OK = 200;
const API_TIMEOUT = 5000;
const MAX_RETRIES = 3;
const JWT_ALGORITHM = "HS256";

const config = {
  algorithm: JWT_ALGORITHM,
  timeout: API_TIMEOUT,
  retries: MAX_RETRIES,
};

if (response.status === HTTP_STATUS_OK) {
  setTimeout(() => {}, API_TIMEOUT);
}
```

**Keuntungan Constants Pattern:**

- Easy maintenance - ganti di satu tempat
- Type safety tanpa `as const`
- Readable dan self-documenting
- Consistent usage di seluruh file

### Single Responsibility File Pattern

```typescript
// ✅ BENAR: Satu file, satu main function
// file: calculateTotal.ts
const TAX_RATE = 0.1;
const SHIPPING_THRESHOLD = 100;

interface CalculateTotalParams {
  subtotal: number;
  shippingCost: number;
  discountAmount?: number;
}

export function calculateTotal({ subtotal, shippingCost, discountAmount = 0 }: CalculateTotalParams): number {
  const discountedSubtotal = subtotal - discountAmount;
  const tax = discountedSubtotal * TAX_RATE;
  const finalShipping = discountedSubtotal >= SHIPPING_THRESHOLD ? 0 : shippingCost;

  return discountedSubtotal + tax + finalShipping;
}
```

### Component Architecture Patterns

#### Single Function per File

- Satu component file = satu main component function
- Helper functions dalam file yang sama jika masih related
- Extract ke separate file jika helper jadi complex

#### Composition over Props Drilling

```typescript
// ✅ BENAR: Composition pattern
export function UserDashboard({ children }: { children: React.ReactNode }) {
  return (
    <div className="dashboard">
      <DashboardHeader />
      {children}
      <DashboardFooter />
    </div>
  );
}

// Usage
<UserDashboard>
  <UserProfile />
  <UserSettings />
</UserDashboard>;
```

#### Custom Hooks Pattern

```typescript
// ✅ BENAR: Single responsibility hook
const RETRY_DELAY = 1000;
const MAX_RETRY_ATTEMPTS = 3;

export function useApiCall<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // Single main function
  const fetchData = useCallback(async () => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);

      const result = await response.json();
      setData(result);
    } catch (err) {
      setError(err instanceof Error ? err.message : "Unknown error");
    } finally {
      setIsLoading(false);
    }
  }, [url]);

  return { data, isLoading, error, refetch: fetchData };
}
```

### Next.js Specific Patterns

#### Server Components First

- Default ke Server Components untuk better performance
- Gunakan "use client" hanya jika benar-benar butuh interactivity
- Leverage Server Actions untuk form handling

#### Data Fetching Pattern

```typescript
// ✅ BENAR: Server Component dengan proper typing
const CACHE_DURATION = 3600; // 1 hour in seconds

interface UserData {
  id: string;
  name: string;
  email: string;
}

export default async function UserPage({ params }: { params: { id: string } }) {
  const userData = await fetchUserById(params.id);

  return (
    <div>
      <UserHeader user={userData} />
      <UserDetails user={userData} />
    </div>
  );
}

async function fetchUserById(id: string): Promise<UserData> {
  const response = await fetch(`${process.env.API_BASE_URL}/users/${id}`, {
    next: { revalidate: CACHE_DURATION },
  });

  if (!response.ok) {
    throw new Error(`Failed to fetch user: ${response.statusText}`);
  }

  return response.json();
}
```

### State Management Strategy

#### Local State First

```typescript
// ✅ BENAR: Local state dengan proper typing
const DEBOUNCE_DELAY = 300

interface SearchState {
  query: string
  results: SearchResult[]
  isSearching: boolean
}

export function SearchComponent() {
  const [searchState, setSearchState] = useState<SearchState>({
    query: '',
    results: [],
    isSearching: false
  })

  const handleSearch = useMemo(() =>
    debounce(async (query: string) => {
      setSearchState(prev => ({ ...prev, isSearching: true }))

      try {
        const results = await searchAPI(query)
        setSearchState(prev => ({ ...prev, results, isSearching: false }))
      } catch (error) {
        console.error('Search failed:', error)
        setSearchState(prev => ({ ...prev, isSearching: false }))
      }
    }, DEBOUNCE_DELAY),
    []
  )

  return (
    // JSX implementation
  )
}
```

#### Global State with Zustand (when needed)

```typescript
// ✅ BENAR: Zustand store dengan TypeScript
const STORAGE_KEY = "user-preferences";

interface UserPreferencesState {
  theme: "light" | "dark";
  language: string;
  notifications: boolean;

  // Actions
  setTheme: (theme: "light" | "dark") => void;
  setLanguage: (language: string) => void;
  toggleNotifications: () => void;
}

export const useUserPreferences = create<UserPreferencesState>()(
  persist(
    (set) => ({
      theme: "light",
      language: "en",
      notifications: true,

      setTheme: (theme) => set({ theme }),
      setLanguage: (language) => set({ language }),
      toggleNotifications: () =>
        set((state) => ({
          notifications: !state.notifications,
        })),
    }),
    { name: STORAGE_KEY }
  )
);
```

### Error Handling Enterprise Patterns

#### Try-Catch with Proper Logging

```typescript
const MAX_RETRY_ATTEMPTS = 3;
const RETRY_DELAY = 1000;

export async function processPayment(paymentData: PaymentData): Promise<PaymentResult> {
  let attempt = 0;

  while (attempt < MAX_RETRY_ATTEMPTS) {
    try {
      const result = await paymentAPI.process(paymentData);

      // Log successful payment
      console.info("Payment processed successfully", {
        paymentId: result.id,
        amount: paymentData.amount,
        timestamp: new Date().toISOString(),
      });

      return result;
    } catch (error) {
      attempt++;

      if (error instanceof PaymentError) {
        // Don't retry for business logic errors
        console.error("Payment business error:", {
          error: error.message,
          code: error.code,
          paymentData: { ...paymentData, cardNumber: "[REDACTED]" },
        });
        throw error;
      }

      if (attempt >= MAX_RETRY_ATTEMPTS) {
        console.error("Payment failed after max retries:", {
          error: error instanceof Error ? error.message : "Unknown error",
          attempts: attempt,
          paymentData: { ...paymentData, cardNumber: "[REDACTED]" },
        });
        throw new Error("Payment processing failed after multiple attempts");
      }

      // Wait before retry
      await new Promise((resolve) => setTimeout(resolve, RETRY_DELAY * attempt));
    }
  }
}
```

### Performance Optimization Rules

#### Memoization Strategy

- `useMemo()` untuk expensive calculations
- `useCallback()` untuk function references dalam dependencies
- `React.memo()` untuk components yang sering re-render
- Avoid inline objects/functions dalam JSX props

#### Bundle Optimization

```typescript
// ✅ BENAR: Dynamic imports dengan proper loading states
const EditUserModal = lazy(() => import("@/components/EditUserModal"));

export function UserManagement() {
  const [showEditModal, setShowEditModal] = useState(false);

  return (
    <div>
      <UserList onEdit={() => setShowEditModal(true)} />

      {showEditModal && (
        <Suspense fallback={<LoadingSpinner />}>
          <EditUserModal onClose={() => setShowEditModal(false)} />
        </Suspense>
      )}
    </div>
  );
}
```

### Environment & Configuration Management

#### Type-Safe Environment Variables

```typescript
// ✅ BENAR: Environment validation dengan Zod
const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qibal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
