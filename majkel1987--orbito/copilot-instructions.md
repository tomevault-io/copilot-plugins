## orbito

> handleSubmit,


Frontend Development Rules for Orbito Platform
🎯 Podstawowe Zasady

Zawsze odpowiadaj w języku polskim - komunikacja z zespołem po polsku
Kod i komentarze zawsze po angielsku - dla zachowania spójności i profesjonalizmu
Limit długości plików: Maksymalnie 500 linii kodu dla komponentów, 300 dla utils
Kolejność działania: Najpierw zainstaluj zależności, potem generuj kod
Transparentność: Jeśli czegoś nie umiesz, powiedz wprost
Human-in-the-loop: Pair programming - razem tworzymy najlepsze rozwiązania
Limit prób: Po 2 nieudanych próbach poproś o pomoc
Kompletność: NIGDY nie zostawiaj TODO w kodzie produkcyjnym

🏗️ Architektura Frontend
Tech Stack

Framework: Next.js 15 z App Router
JavaScript: ES6+ (bez TypeScript na tym etapie)
Styling: Tailwind CSS 3.4
UI Components: shadcn/ui
State Management: Zustand dla global state, React hooks dla local state
Forms: React Hook Form + Zod (z JS schema)
Data Fetching: TanStack Query (React Query) v5
API Client: Axios z interceptorami
Date/Time: date-fns
Icons: Lucide React
Charts: Recharts
Tables: TanStack Table
Testing: Vitest + React Testing Library (gdy będzie potrzebne)
E2E: Playwright (gdy będzie potrzebne)

Struktura Katalogów
src/
├── app/ # Next.js App Router
│ ├── (auth)/ # Publiczne strony (login, register)
│ ├── (dashboard)/ # Chronione strony
│ │ ├── provider/ # Panel providera
│ │ ├── client/ # Panel klienta
│ │ └── admin/ # Panel admina
│ ├── api/ # API routes (jeśli potrzebne)
│ ├── layout.jsx # Root layout
│ └── page.jsx # Landing page
├── components/
│ ├── ui/ # shadcn/ui komponenty
│ ├── common/ # Współdzielone komponenty
│ ├── features/ # Feature-specific komponenty
│ │ ├── providers/
│ │ ├── clients/
│ │ ├── services/
│ │ └── payments/
│ └── layouts/ # Layout komponenty
├── hooks/ # Custom React hooks
├── lib/ # Biblioteki i utilities
│ ├── api/ # API client i endpoints
│ ├── auth/ # Autoryzacja i JWT
│ ├── utils/ # Helper functions
│ └── validations/ # Zod schemas
├── stores/ # Zustand stores
├── styles/ # Globalne style
└── config/ # Konfiguracja
🎨 Styling i UI
Tailwind CSS Konwencje
jsx// ✅ DOBRE - czytelne klasy w logicznej kolejności

<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow-md hover:shadow-lg transition-shadow">

// ❌ ZŁE - chaotyczne, długie stringi

<div className="hover:shadow-lg p-4 flex bg-white transition-shadow rounded-lg items-center shadow-md justify-between">
Kolejność Klas Tailwind

Positioning: relative, absolute, fixed
Display: flex, grid, block
Flex/Grid properties: items-center, justify-between
Sizing: w-full, h-screen
Spacing: p-4, m-2, gap-4
Background: bg-white, bg-gradient-to-r
Border: border, rounded-lg
Effects: shadow-md, opacity-50
Transitions: transition-all, duration-200
States: hover:, focus:, disabled:

Komponenty UI (shadcn/ui)
jsx// Importuj z aliasu
import { Button } from "@/components/ui/button"
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"
import {
Dialog,
DialogContent,
DialogDescription,
DialogHeader,
DialogTitle,
} from "@/components/ui/dialog"

// Używaj wariantów
<Button variant="destructive" size="lg">
Delete Provider
</Button>

// Komponuj z cn() dla custom stylów
import { cn } from "@/lib/utils"

<Card className={cn("hover:shadow-lg transition-shadow", customClass)}>
Dark Mode
jsx// Wszystkie komponenty muszą wspierać dark mode

<div className="bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100">
  <p className="text-gray-600 dark:text-gray-400">Secondary text</p>
</div>
📝 JavaScript Standards
JSDoc dla Dokumentacji
javascript/**
 * Creates a new provider
 * @param {Object} data - Provider data
 * @param {string} data.name - Provider name
 * @param {string} data.subdomain - Provider subdomain
 * @param {string} data.email - Provider email
 * @returns {Promise<Object>} Created provider object
 */
export async function createProvider(data) {
  // implementation
}
PropTypes dla Walidacji Props
jsximport PropTypes from 'prop-types'

function ProviderCard({ provider, onEdit, onDelete }) {
return (
<Card>
{/_ content _/}
</Card>
)
}

ProviderCard.propTypes = {
provider: PropTypes.shape({
id: PropTypes.string.isRequired,
name: PropTypes.string.isRequired,
subdomain: PropTypes.string.isRequired,
status: PropTypes.oneOf(['pending', 'active', 'inactive']).isRequired,
}).isRequired,
onEdit: PropTypes.func,
onDelete: PropTypes.func,
}

ProviderCard.defaultProps = {
onEdit: undefined,
onDelete: undefined,
}

export default ProviderCard
Zod Schemas (w JavaScript)
javascript// lib/validations/provider.js
import { z } from 'zod'

export const createProviderSchema = z.object({
name: z.string()
.min(2, 'Name must be at least 2 characters')
.max(100, 'Name must be less than 100 characters'),
subdomain: z.string()
.min(3, 'Subdomain must be at least 3 characters')
.regex(/^[a-z0-9-]+$/, 'Only lowercase letters, numbers and hyphens'),
email: z.string().email('Invalid email address'),
phone: z.string().optional(),
})
Nowoczesny JavaScript ES6+
javascript// ✅ DOBRE - destructuring, arrow functions, async/await
const { name, email } = provider
const filteredProviders = providers.filter(p => p.status === 'active')
const data = await fetchProviders()

// ✅ DOBRE - spread operator, template literals
const updatedProvider = { ...provider, status: 'active' }
const url = `https://${subdomain}.orbito.com`

// ✅ DOBRE - optional chaining, nullish coalescing
const displayName = provider?.name ?? 'Unknown'
const count = providers?.length ?? 0

// ❌ ZŁE - var, function expressions
var x = 10
var getName = function() { return this.name }
⚛️ React Best Practices
Komponenty Funkcyjne
jsx// ✅ DOBRE - Functional component z PropTypes
function ProviderCard({ provider, onEdit, onDelete }) {
return (
<Card>
<CardHeader>
<CardTitle>{provider.name}</CardTitle>
</CardHeader>
<CardContent>

<p>{provider.subdomain}.orbito.com</p>
</CardContent>
</Card>
)
}

ProviderCard.propTypes = {
provider: PropTypes.object.isRequired,
onEdit: PropTypes.func,
onDelete: PropTypes.func,
}

export default ProviderCard

// ❌ ZŁE - Class components (bez potrzeby)
class ProviderCard extends React.Component {
render() {
return <div>{this.props.provider.name}</div>
}
}
Custom Hooks
javascript// hooks/use-providers.js
import { useQuery } from '@tanstack/react-query'
import { fetchProviders } from '@/lib/api/providers'

export function useProviders() {
return useQuery({
queryKey: ['providers'],
queryFn: fetchProviders,
staleTime: 5 _ 60 _ 1000, // 5 minutes
})
}

// hooks/use-debounce.js
import { useState, useEffect } from 'react'

export function useDebounce(value, delay) {
const [debouncedValue, setDebouncedValue] = useState(value)

useEffect(() => {
const timer = setTimeout(() => setDebouncedValue(value), delay)
return () => clearTimeout(timer)
}, [value, delay])

return debouncedValue
}
Server vs Client Components
jsx// Server Component (domyślnie w Next.js 15)
// app/(dashboard)/providers/page.jsx
export default async function ProvidersPage() {
const providers = await fetchProviders() // Server-side fetch

return (

<div>
<ProvidersList providers={providers} />
</div>
)
}

// Client Component (użyj "use client" gdy potrzebne)
// components/features/providers/providers-list.jsx
"use client"

import { useState } from 'react'

export function ProvidersList({ providers }) {
const [search, setSearch] = useState("")
// interaktywność, hooks, browser APIs

return (

<div>
{/_ interactive content _/}
</div>
)
}

ProvidersList.propTypes = {
providers: PropTypes.array.isRequired,
}
Optymalizacja Wydajności
jsximport { memo, useMemo, useCallback } from 'react'

// Memoizacja komponentów
const ExpensiveComponent = memo(function ExpensiveComponent({ data }) {
return <div>{/_ heavy render _/}</div>
})

ExpensiveComponent.propTypes = {
data: PropTypes.object.isRequired,
}

// useMemo dla kosztownych obliczeń
function FilteredList({ data, search }) {
const filteredData = useMemo(
() => data.filter(item => item.name.includes(search)),
[data, search]
)

return <div>{/_ render filtered data _/}</div>
}

// useCallback dla funkcji przekazywanych jako props
function ParentComponent() {
const handleClick = useCallback((id) => {
// handle click
}, [])

return <ChildComponent onClick={handleClick} />
}

// Lazy loading dla route splitting
import { lazy } from 'react'

const AdminPanel = lazy(() => import('@/components/admin-panel'))
🔄 State Management
Zustand Stores
javascript// stores/auth-store.js
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

export const useAuthStore = create(
persist(
(set) => ({
user: null,
token: null,
isAuthenticated: false,

      login: (user, token) => set({
        user,
        token,
        isAuthenticated: true
      }),

      logout: () => set({
        user: null,
        token: null,
        isAuthenticated: false
      }),

      updateUser: (userData) => set((state) => ({
        user: { ...state.user, ...userData }
      })),
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({ token: state.token }), // Persist tylko token
    }

)
)
Local State Patterns
jsximport { useState, useReducer } from 'react'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'

// useState dla prostego state
function SimpleComponent() {
const [isOpen, setIsOpen] = useState(false)
const [count, setCount] = useState(0)

return <div>{/_ content _/}</div>
}

// useReducer dla złożonego state
const initialState = { count: 0, step: 1 }

function reducer(state, action) {
switch (action.type) {
case 'increment':
return { ...state, count: state.count + state.step }
case 'decrement':
return { ...state, count: state.count - state.step }
case 'setStep':
return { ...state, step: action.payload }
default:
return state
}
}

function ComplexComponent() {
const [state, dispatch] = useReducer(reducer, initialState)

return <div>{/_ content _/}</div>
}

// React Hook Form dla formularzy
function FormComponent() {
const { register, handleSubmit, formState: { errors } } = useForm({
resolver: zodResolver(schema)
})

const onSubmit = (data) => {
console.log(data)
}

return <form onSubmit={handleSubmit(onSubmit)}>{/_ fields _/}</form>
}
📡 Data Fetching
TanStack Query Setup
javascript// lib/api/query-client.js
import { QueryClient } from '@tanstack/react-query'

export const queryClient = new QueryClient({
defaultOptions: {
queries: {
staleTime: 5 _ 60 _ 1000, // 5 minutes
retry: 1,
refetchOnWindowFocus: false,
},
},
})
API Client z Axios
javascript// lib/api/client.js
import axios from 'axios'
import { useAuthStore } from '@/stores/auth-store'

const apiClient = axios.create({
baseURL: process.env.NEXT_PUBLIC_API_URL,
headers: {
'Content-Type': 'application/json',
},
})

// Request interceptor dla auth
apiClient.interceptors.request.use((config) => {
const token = useAuthStore.getState().token
if (token) {
config.headers.Authorization = `Bearer ${token}`
}
return config
})

// Response interceptor dla error handling
apiClient.interceptors.response.use(
(response) => response,
async (error) => {
if (error.response?.status === 401) {
useAuthStore.getState().logout()
window.location.href = '/login'
}
return Promise.reject(error)
}
)

export default apiClient
Query i Mutation Hooks
javascript// hooks/api/use-providers.js
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { toast } from 'sonner'
import \* as providersApi from '@/lib/api/providers'

export function useProviders(params) {
return useQuery({
queryKey: ['providers', params],
queryFn: () => providersApi.getAll(params),
})
}

export function useCreateProvider() {
const queryClient = useQueryClient()

return useMutation({
mutationFn: providersApi.create,
onSuccess: () => {
queryClient.invalidateQueries({ queryKey: ['providers'] })
toast.success('Provider created successfully')
},
onError: (error) => {
toast.error(error.response?.data?.message || 'Failed to create provider')
},
})
}
📋 Forms
React Hook Form + Zod
jsx// components/features/providers/create-provider-form.jsx
"use client"

import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { createProviderSchema } from '@/lib/validations/provider'
import { useCreateProvider } from '@/hooks/api/use-providers'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'

export function CreateProviderForm() {
const {
register,
handleSubmit,
formState: { errors, isSubmitting },
} = useForm({
resolver: zodResolver(createProviderSchema),
})

const { mutate: createProvider } = useCreateProvider()

const onSubmit = async (data) => {
createProvider(data)
}

return (

<form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
<div>
<Label htmlFor="name">Name</Label>
<Input
id="name"
{...register('name')}
className={errors.name ? 'border-red-500' : ''}
/>
{errors.name && (
<p className="text-sm text-red-500 mt-1">{errors.name.message}</p>
)}
</div>

      <div>
        <Label htmlFor="subdomain">Subdomain</Label>
        <Input
          id="subdomain"
          {...register('subdomain')}
          className={errors.subdomain ? 'border-red-500' : ''}
        />
        {errors.subdomain && (
          <p className="text-sm text-red-500 mt-1">{errors.subdomain.message}</p>
        )}
      </div>

      <Button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Creating...' : 'Create Provider'}
      </Button>
    </form>

)
}
🔐 Autoryzacja i Bezpieczeństwo
JWT Token Management
javascript// lib/auth/token.js

/\*\*

- Manages JWT tokens for authentication
  \*/
  export const tokenManager = {
  /\*\*
  - Gets token from localStorage
  - @returns {string|null} JWT token or null
    \*/
    getToken: () => {
    if (typeof window === 'undefined') return null
    return localStorage.getItem('auth_token')
    },

/\*\*

- Saves token to localStorage
- @param {string} token - JWT token to save
  \*/
  setToken: (token) => {
  if (typeof window === 'undefined') return
  localStorage.setItem('auth_token', token)
  },

/\*\*

- Removes token from localStorage
  \*/
  removeToken: () => {
  if (typeof window === 'undefined') return
  localStorage.removeItem('auth_token')
  },

/\*\*

- Decodes JWT token
- @param {string} token - JWT token to decode
- @returns {Object|null} Decoded payload or null
  \*/
  decodeToken: (token) => {
  try {
  return JSON.parse(atob(token.split('.')[1]))
  } catch {
  return null
  }
  },

/\*\*

- Checks if token is expired
- @param {string} token - JWT token to check
- @returns {boolean} True if expired
  _/
  isExpired: (token) => {
  const payload = tokenManager.decodeToken(token)
  if (!payload) return true
  return Date.now() >= payload.exp _ 1000
  }
  }
  Protected Routes
  javascript// middleware.js
  import { NextResponse } from 'next/server'

/\*\*

- Middleware for route protection
- @param {import('next/server').NextRequest} request
  \*/
  export function middleware(request) {
  const token = request.cookies.get('auth_token')?.value
  const isAuthPage = request.nextUrl.pathname.startsWith('/login')

if (!token && !isAuthPage) {
return NextResponse.redirect(new URL('/login', request.url))
}

if (token && isAuthPage) {
return NextResponse.redirect(new URL('/dashboard', request.url))
}

return NextResponse.next()
}

export const config = {
matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
}
Role-Based Access Control
jsx// components/common/role-guard.jsx
import PropTypes from 'prop-types'
import { useAuthStore } from '@/stores/auth-store'

export function RoleGuard({ allowedRoles, children, fallback }) {
const user = useAuthStore(state => state.user)

if (!user || !allowedRoles.includes(user.role)) {
return fallback || <div>Access Denied</div>
}

return <>{children}</>
}

RoleGuard.propTypes = {
allowedRoles: PropTypes.arrayOf(
PropTypes.oneOf(['provider', 'client', 'admin'])
).isRequired,
children: PropTypes.node.isRequired,
fallback: PropTypes.node,
}

// Użycie
<RoleGuard allowedRoles={['provider', 'admin']}>
<ProviderSettings />
</RoleGuard>
🚀 Performance
Next.js Optimizations
jsx// Image Optimization
import Image from 'next/image'

<Image
src="/provider-logo.png"
alt="Provider Logo"
width={200}
height={200}
priority // dla above-the-fold images
placeholder="blur" // z blurDataURL
/>

// Font Optimization
import { Inter } from 'next/font/google'

const inter = Inter({
subsets: ['latin'],
variable: '--font-inter',
})

// Link Prefetching
import Link from 'next/link'

<Link href="/providers" prefetch>
  View Providers
</Link>
Bundle Size Optimization
javascript// Dynamic imports dla heavy components
import dynamic from 'next/dynamic'
import { Skeleton } from '@/components/ui/skeleton'

const HeavyChart = dynamic(() => import('@/components/charts/heavy-chart'), {
loading: () => <Skeleton className="h-64" />,
ssr: false,
})

// Tree shaking imports
import { format } from 'date-fns/format' // ✅ Specific import
// import \* as dateFns from 'date-fns' // ❌ Imports everything
React Query Optimizations
javascript// Prefetching
await queryClient.prefetchQuery({
queryKey: ['provider', id],
queryFn: () => fetchProvider(id),
})

// Optimistic Updates
const mutation = useMutation({
mutationFn: updateProvider,
onMutate: async (newData) => {
await queryClient.cancelQueries(['provider', id])
const previousData = queryClient.getQueryData(['provider', id])
queryClient.setQueryData(['provider', id], newData)
return { previousData }
},
onError: (err, newData, context) => {
queryClient.setQueryData(['provider', id], context.previousData)
},
onSettled: () => {
queryClient.invalidateQueries(['provider', id])
},
})
📦 Environment Variables
.env.local
env# API Configuration
NEXT_PUBLIC_API_URL=http://localhost:5000/api
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Feature Flags

NEXT_PUBLIC_ENABLE_PAYMENTS=true
NEXT_PUBLIC_ENABLE_ANALYTICS=false

# Third Party

NEXT*PUBLIC_STRIPE_PUBLIC_KEY=pk_test*...
Dostęp do Environment Variables
javascript// config/env.js

/\*\*

- Application environment configuration
  \*/
  export const env = {
  apiUrl: process.env.NEXT_PUBLIC_API_URL,
  appUrl: process.env.NEXT_PUBLIC_APP_URL,
  stripePublicKey: process.env.NEXT_PUBLIC_STRIPE_PUBLIC_KEY,
  enablePayments: process.env.NEXT_PUBLIC_ENABLE_PAYMENTS === 'true',
  enableAnalytics: process.env.NEXT_PUBLIC_ENABLE_ANALYTICS === 'true',
  }

// Walidacja środowiska przy starcie
if (!env.apiUrl) {
throw new Error('NEXT*PUBLIC_API_URL is not defined')
}
🎯 Code Quality
ESLint Configuration
json// .eslintrc.json
{
"extends": [
"next/core-web-vitals",
"prettier"
],
"rules": {
"no-unused-vars": "error",
"no-console": ["warn", { "allow": ["warn", "error"] }],
"react-hooks/rules-of-hooks": "error",
"react-hooks/exhaustive-deps": "warn",
"prefer-const": "error",
"no-var": "error"
}
}
Prettier Configuration
json// .prettierrc
{
"semi": false,
"singleQuote": true,
"tabWidth": 2,
"trailingComma": "es5",
"printWidth": 100,
"arrowParens": "always"
}
Pre-commit Hooks (opcjonalnie)
json// package.json
{
"husky": {
"hooks": {
"pre-commit": "lint-staged"
}
},
"lint-staged": {
"*.{js,jsx}": ["eslint --fix", "prettier --write"],
"\_.{css,scss}": ["prettier --write"]
}
}
📊 Monitoring i Analytics
Error Boundary
jsx// components/common/error-boundary.jsx
"use client"

import { Component } from 'react'
import PropTypes from 'prop-types'
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card'
import { Button } from '@/components/ui/button'

export class ErrorBoundary extends Component {
constructor(props) {
super(props)
this.state = { hasError: false, error: null }
}

static getDerivedStateFromError(error) {
return { hasError: true, error }
}

componentDidCatch(error, errorInfo) {
console.error('Error caught by boundary:', error, errorInfo)
// Send to monitoring service (Sentry, LogRocket)
}

render() {
if (this.state.hasError) {
return (

<div className="min-h-screen flex items-center justify-center p-4">
<Card className="max-w-md w-full">
<CardHeader>
<CardTitle>Oops! Something went wrong</CardTitle>
</CardHeader>
<CardContent className="space-y-4">
<p className="text-sm text-gray-600">
We're sorry for the inconvenience. Please try reloading the page.
</p>
<Button
onClick={() => window.location.reload()}
className="w-full" >
Reload Page
</Button>
</CardContent>
</Card>
</div>
)
}

    return this.props.children

}
}

ErrorBoundary.propTypes = {
children: PropTypes.node.isRequired,
}
🚨 Do Unikania
Anti-Patterns

❌ var - używaj const i let
❌ Brak PropTypes - zawsze definiuj propsy
❌ Inline styles - używaj Tailwind classes
❌ Direct DOM manipulation - używaj React state
❌ Synchronous localStorage w render - używaj useEffect
❌ Large components (>300 lines) - dziel na mniejsze
❌ Business logic w komponentach - przenieś do hooks/utils
❌ Hardcoded URLs - używaj environment variables
❌ Missing loading states - zawsze pokazuj feedback
❌ Missing error handling - każdy request może fail
❌ Unoptimized images - używaj Next.js Image
❌ Mutowanie state bezpośrednio - używaj immutable updates

Security Anti-Patterns

❌ Storing sensitive data in localStorage
❌ Exposing API keys in client code
❌ Missing CSRF protection
❌ Direct HTML injection (dangerouslySetInnerHTML)
❌ Unvalidated user input
❌ Missing rate limiting on API calls

📋 Code Review Checklist
Funkcjonalność

Kod działa bez błędów
Feature działa zgodnie z wymaganiami
Obsługa edge cases
PropTypes są poprawnie zdefiniowane

UI/UX

Responsive design (mobile, tablet, desktop)
Dark mode support
Loading states
Error states
Accessibility (ARIA labels, keyboard navigation)
Animations są smooth (60fps)

Performance

Images są zoptymalizowane
Lazy loading gdzie możliwe
Memoization dla expensive operations
Bundle size nie wzrósł znacząco

Code Quality

PropTypes są dokładne i kompletne
JSDoc comments dla funkcji i komponentów
Komponenty są reusable
Kod jest DRY (Don't Repeat Yourself)
Nazwy są opisowe i zrozumiałe
Używane są nowoczesne ES6+ features

Security

User input jest walidowany
API keys nie są exposed
XSS protection
Sensitive data nie jest w localStorage

🔄 Git Workflow
Branch Naming
bashfeature/provider-dashboard
bugfix/login-validation
hotfix/payment-error
refactor/api-client
Commit Messages (Conventional Commits)
bashfeat: add provider dashboard
fix: resolve login validation issue
refactor: optimize API client
style: format provider components
docs: update frontend documentation
chore: update dependencies
📚 Dodatkowe Zasoby

Next.js 15 Docs: https://nextjs.org/docs
shadcn/ui: https://ui.shadcn.com
Tailwind CSS: https://tailwindcss.com/docs
TanStack Query: https://tanstack.com/query
React Hook Form: https

- **React Hook Form**: https://react-hook-form.com
- **Zod**: https://zod.dev
- **Zustand**: https://zustand-demo.pmnd.rs
- **Axios**: https://axios-http.com/docs/intro
- **PropTypes**: https://legacy.reactjs.org/docs/typechecking-with-proptypes.html

---

## 🎓 Konwencje Nazewnicze

### Pliki i Foldery

```
// Komponenty - PascalCase
ProviderCard.jsx
CreateProviderForm.jsx

// Hooks - camelCase z prefiksem 'use'
use-providers.js
use-debounce.js

// Utils i helpers - camelCase
api-client.js
format-currency.js

// Stores - camelCase z sufixem 'store'
auth-store.js
providers-store.js

// Validations - camelCase z sufixem schema/validation
provider-schema.js
user-validation.js
```

### Zmienne i Funkcje

```javascript
// ✅ DOBRE - opisowe, camelCase
const isAuthenticated = true;
const providerCount = providers.length;
const handleSubmit = () => {};
const fetchProviderData = async () => {};

// ❌ ZŁE - niejasne, źle nazwane
const flag = true;
const cnt = providers.length;
const handle = () => {};
const getData = async () => {};
```

### Stałe

```javascript
// Constants - UPPER_SNAKE_CASE
const API_BASE_URL = "https://api.orbito.com";
const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
const DEBOUNCE_DELAY = 300;

// Enums/Objects jako stałe
const USER_ROLES = {
  PROVIDER: "provider",
  CLIENT: "client",
  ADMIN: "admin",
};

const PROVIDER_STATUS = {
  PENDING: "pending",
  ACTIVE: "active",
  INACTIVE: "inactive",
  SUSPENDED: "suspended",
};
```

### Boolean Naming

```javascript
// Prefiksy: is, has, can, should
const isLoading = true;
const hasPermission = false;
const canEdit = true;
const shouldRefresh = false;

// ✅ DOBRE
if (isAuthenticated && canEdit) {
  // logic
}

// ❌ ZŁE
if (authenticated && edit) {
  // logic
}
```

## 🛠️ Pomocnicze Utilities

### API Helpers

```javascript
// lib/api/helpers.js

/**
 * Handles API errors and returns user-friendly messages
 * @param {Error} error - Axios error object
 * @returns {string} User-friendly error message
 */
export function handleApiError(error) {
  if (error.response) {
    // Server responded with error
    const message = error.response.data?.message;
    const status = error.response.status;

    if (status === 401) return "Unauthorized. Please login again.";
    if (status === 403)
      return "You do not have permission to perform this action.";
    if (status === 404) return "Resource not found.";
    if (status === 422) return message || "Validation error.";
    if (status >= 500) return "Server error. Please try again later.";

    return message || "An error occurred.";
  }

  if (error.request) {
    // Request made but no response
    return "Network error. Please check your connection.";
  }

  return error.message || "An unexpected error occurred.";
}

/**
 * Builds query string from params object
 * @param {Object} params - Query parameters
 * @returns {string} Query string
 */
export function buildQueryString(params) {
  const filtered = Object.entries(params)
    .filter(
      ([_, value]) => value !== undefined && value !== null && value !== ""
    )
    .map(
      ([key, value]) =>
        `${encodeURIComponent(key)}=${encodeURIComponent(value)}`
    )
    .join("&");

  return filtered ? `?${filtered}` : "";
}

/**
 * Creates headers with authentication
 * @returns {Object} Headers object
 */
export function createAuthHeaders() {
  const token =
    typeof window !== "undefined" ? localStorage.getItem("auth_token") : null;

  return {
    "Content-Type": "application/json",
    ...(token && { Authorization: `Bearer ${token}` }),
  };
}
```

### Format Helpers

```javascript
// lib/utils/format.js
import { format, formatDistance, parseISO } from "date-fns";
import { pl } from "date-fns/locale";

/**
 * Formats currency value
 * @param {number} amount - Amount in cents
 * @param {string} currency - Currency code (default: PLN)
 * @returns {string} Formatted currency string
 */
export function formatCurrency(amount, currency = "PLN") {
  return new Intl.NumberFormat("pl-PL", {
    style: "currency",
    currency,
  }).format(amount / 100);
}

/**
 * Formats date to readable string
 * @param {string|Date} date - Date to format
 * @param {string} formatStr - Format string (default: 'dd.MM.yyyy')
 * @returns {string} Formatted date string
 */
export function formatDate(date, formatStr = "dd.MM.yyyy") {
  const dateObj = typeof date === "string" ? parseISO(date) : date;
  return format(dateObj, formatStr, { locale: pl });
}

/**
 * Formats date to relative time (e.g., "2 hours ago")
 * @param {string|Date} date - Date to format
 * @returns {string} Relative time string
 */
export function formatRelativeTime(date) {
  const dateObj = typeof date === "string" ? parseISO(date) : date;
  return formatDistance(dateObj, new Date(), {
    addSuffix: true,
    locale: pl,
  });
}

/**
 * Formats phone number
 * @param {string} phone - Phone number
 * @returns {string} Formatted phone number
 */
export function formatPhone(phone) {
  if (!phone) return "";
  const cleaned = phone.replace(/\D/g, "");

  // Polish format: +48 123 456 789
  if (cleaned.startsWith("48") && cleaned.length === 11) {
    return `+${cleaned.slice(0, 2)} ${cleaned.slice(2, 5)} ${cleaned.slice(
      5,
      8
    )} ${cleaned.slice(8)}`;
  }

  // Default format: XXX-XXX-XXX
  return cleaned.replace(/(\d{3})(\d{3})(\d{3})/, "$1-$2-$3");
}

/**
 * Truncates text with ellipsis
 * @param {string} text - Text to truncate
 * @param {number} maxLength - Maximum length
 * @returns {string} Truncated text
 */
export function truncate(text, maxLength = 50) {
  if (!text) return "";
  if (text.length <= maxLength) return text;
  return `${text.slice(0, maxLength)}...`;
}

/**
 * Formats file size
 * @param {number} bytes - Size in bytes
 * @returns {string} Formatted file size
 */
export function formatFileSize(bytes) {
  if (bytes === 0) return "0 B";

  const k = 1024;
  const sizes = ["B", "KB", "MB", "GB"];
  const i = Math.floor(Math.log(bytes) / Math.log(k));

  return `${parseFloat((bytes / Math.pow(k, i)).toFixed(2))} ${sizes[i]}`;
}
```

### Validation Helpers

```javascript
// lib/utils/validation.js

/**
 * Validates Polish NIP (tax identification number)
 * @param {string} nip - NIP to validate
 * @returns {boolean} True if valid
 */
export function isValidNIP(nip) {
  if (!nip) return false;

  const cleaned = nip.replace(/[\s-]/g, "");
  if (!/^\d{10}$/.test(cleaned)) return false;

  const weights = [6, 5, 7, 2, 3, 4, 5, 6, 7];
  const sum = cleaned
    .split("")
    .slice(0, 9)
    .reduce((acc, digit, index) => acc + parseInt(digit) * weights[index], 0);

  const checksum = sum % 11;
  return checksum === parseInt(cleaned[9]);
}

/**
 * Validates Polish REGON number
 * @param {string} regon - REGON to validate
 * @returns {boolean} True if valid
 */
export function isValidREGON(regon) {
  if (!regon) return false;

  const cleaned = regon.replace(/[\s-]/g, "");
  if (!/^\d{9}$/.test(cleaned) && !/^\d{14}$/.test(cleaned)) return false;

  return true; // Simplified - full validation would include checksum
}

/**
 * Validates email format
 * @param {string} email - Email to validate
 * @returns {boolean} True if valid
 */
export function isValidEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}

/**
 * Validates subdomain format
 * @param {string} subdomain - Subdomain to validate
 * @returns {boolean} True if valid
 */
export function isValidSubdomain(subdomain) {
  const subdomainRegex = /^[a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?$/;
  return subdomainRegex.test(subdomain);
}

/**
 * Validates password strength
 * @param {string} password - Password to validate
 * @returns {Object} Validation result with strength level
 */
export function validatePasswordStrength(password) {
  const result = {
    isValid: false,
    strength: "weak",
    feedback: [],
  };

  if (password.length < 8) {
    result.feedback.push("Password must be at least 8 characters");
  }

  if (!/[a-z]/.test(password)) {
    result.feedback.push("Password must contain lowercase letter");
  }

  if (!/[A-Z]/.test(password)) {
    result.feedback.push("Password must contain uppercase letter");
  }

  if (!/[0-9]/.test(password)) {
    result.feedback.push("Password must contain number");
  }

  if (!/[!@#$%^&*]/.test(password)) {
    result.feedback.push("Password must contain special character");
  }

  const strengthScore = 5 - result.feedback.length;

  if (strengthScore >= 4) result.strength = "strong";
  else if (strengthScore >= 3) result.strength = "medium";
  else result.strength = "weak";

  result.isValid = result.feedback.length === 0;

  return result;
}
```

### Array & Object Helpers

```javascript
// lib/utils/helpers.js

/**
 * Groups array of objects by key
 * @param {Array} array - Array to group
 * @param {string} key - Key to group by
 * @returns {Object} Grouped object
 */
export function groupBy(array, key) {
  return array.reduce((result, item) => {
    const group = item[key];
    if (!result[group]) result[group] = [];
    result[group].push(item);
    return result;
  }, {});
}

/**
 * Removes duplicates from array
 * @param {Array} array - Array with duplicates
 * @param {string} key - Key to check for uniqueness (for objects)
 * @returns {Array} Array without duplicates
 */
export function uniqueBy(array, key) {
  if (!key) return [...new Set(array)];

  const seen = new Set();
  return array.filter((item) => {
    const value = item[key];
    if (seen.has(value)) return false;
    seen.add(value);
    return true;
  });
}

/**
 * Deep clones an object
 * @param {Object} obj - Object to clone
 * @returns {Object} Cloned object
 */
export function deepClone(obj) {
  return JSON.parse(JSON.stringify(obj));
}

/**
 * Checks if object is empty
 * @param {Object} obj - Object to check
 * @returns {boolean} True if empty
 */
export function isEmpty(obj) {
  if (obj == null) return true;
  if (Array.isArray(obj)) return obj.length === 0;
  if (typeof obj === "object") return Object.keys(obj).length === 0;
  return false;
}

/**
 * Safely gets nested property value
 * @param {Object} obj - Object to get value from
 * @param {string} path - Dot-separated path (e.g., 'user.address.city')
 * @param {*} defaultValue - Default value if path not found
 * @returns {*} Value or default
 */
export function get(obj, path, defaultValue = undefined) {
  const keys = path.split(".");
  let result = obj;

  for (const key of keys) {
    if (result?.[key] === undefined) return defaultValue;
    result = result[key];
  }

  return result ?? defaultValue;
}

/**
 * Sorts array of objects by key
 * @param {Array} array - Array to sort
 * @param {string} key - Key to sort by
 * @param {string} order - Sort order ('asc' or 'desc')
 * @returns {Array} Sorted array
 */
export function sortBy(array, key, order = "asc") {
  return [...array].sort((a, b) => {
    const valueA = a[key];
    const valueB = b[key];

    if (valueA < valueB) return order === "asc" ? -1 : 1;
    if (valueA > valueB) return order === "asc" ? 1 : -1;
    return 0;
  });
}
```

## 🎨 Reusable Component Patterns

### Loading States

```jsx
// components/common/loading-state.jsx
import PropTypes from "prop-types";
import { Loader2 } from "lucide-react";

export function LoadingState({ message = "Loading...", className = "" }) {
  return (
    <div
      className={`flex flex-col items-center justify-center p-8 ${className}`}
    >
      <Loader2 className="w-8 h-8 animate-spin text-primary mb-2" />
      <p className="text-sm text-gray-600 dark:text-gray-400">{message}</p>
    </div>
  );
}

LoadingState.propTypes = {
  message: PropTypes.string,
  className: PropTypes.string,
};

// Skeleton Loader
import { Skeleton } from "@/components/ui/skeleton";

export function CardSkeleton() {
  return (
    <div className="p-6 space-y-4 border rounded-lg">
      <Skeleton className="h-6 w-2/3" />
      <Skeleton className="h-4 w-full" />
      <Skeleton className="h-4 w-4/5" />
      <div className="flex gap-2">
        <Skeleton className="h-8 w-20" />
        <Skeleton className="h-8 w-20" />
      </div>
    </div>
  );
}
```

### Empty States

```jsx
// components/common/empty-state.jsx
import PropTypes from "prop-types";
import { FileX } from "lucide-react";
import { Button } from "@/components/ui/button";

export function EmptyState({
  icon: Icon = FileX,
  title = "No data found",
  description,
  action,
  actionLabel,
}) {
  return (
    <div className="flex flex-col items-center justify-center p-12 text-center">
      <div className="w-16 h-16 rounded-full bg-gray-100 dark:bg-gray-800 flex items-center justify-center mb-4">
        <Icon className="w-8 h-8 text-gray-400" />
      </div>
      <h3 className="text-lg font-semibold mb-2">{title}</h3>
      {description && (
        <p className="text-sm text-gray-600 dark:text-gray-400 mb-6 max-w-sm">
          {description}
        </p>
      )}
      {action && actionLabel && <Button onClick={action}>{actionLabel}</Button>}
    </div>
  );
}

EmptyState.propTypes = {
  icon: PropTypes.elementType,
  title: PropTypes.string,
  description: PropTypes.string,
  action: PropTypes.func,
  actionLabel: PropTypes.string,
};
```

### Error States

```jsx
// components/common/error-state.jsx
import PropTypes from "prop-types";
import { AlertCircle } from "lucide-react";
import { Button } from "@/components/ui/button";
import { Alert, AlertDescription, AlertTitle } from "@/components/ui/alert";

export function ErrorState({
  title = "Something went wrong",
  message,
  onRetry,
}) {
  return (
    <div className="p-6">
      <Alert variant="destructive">
        <AlertCircle className="h-4 w-4" />
        <AlertTitle>{title}</AlertTitle>
        {message && <AlertDescription>{message}</AlertDescription>}
      </Alert>
      {onRetry && (
        <div className="mt-4 flex justify-center">
          <Button onClick={onRetry} variant="outline">
            Try Again
          </Button>
        </div>
      )}
    </div>
  );
}

ErrorState.propTypes = {
  title: PropTypes.string,
  message: PropTypes.string,
  onRetry: PropTypes.func,
};
```

### Confirmation Dialog

```jsx
// components/common/confirmation-dialog.jsx
import PropTypes from "prop-types";
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
} from "@/components/ui/alert-dialog";

export function ConfirmationDialog({
  open,
  onOpenChange,
  title,
  description,
  onConfirm,
  confirmLabel = "Confirm",
  cancelLabel = "Cancel",
  variant = "default",
}) {
  return (
    <AlertDialog open={open} onOpenChange={onOpenChange}>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>{title}</AlertDialogTitle>
          <AlertDialogDescription>{description}</AlertDialogDescription>
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel>{cancelLabel}</AlertDialogCancel>
          <AlertDialogAction
            onClick={onConfirm}
            className={
              variant === "destructive" ? "bg-red-600 hover:bg-red-700" : ""
            }
          >
            {confirmLabel}
          </AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  );
}

ConfirmationDialog.propTypes = {
  open: PropTypes.bool.isRequired,
  onOpenChange: PropTypes.func.isRequired,
  title: PropTypes.string.isRequired,
  description: PropTypes.string,
  onConfirm: PropTypes.func.isRequired,
  confirmLabel: PropTypes.string,
  cancelLabel: PropTypes.string,
  variant: PropTypes.oneOf(["default", "destructive"]),
};
```

## 🌟 Kluczowe Zasady do Zapamiętania

1. **Mobile First**: Projektuj najpierw dla mobile, potem skaluj
2. **PropTypes Everywhere**: Każdy komponent musi mieć PropTypes
3. **JSDoc Comments**: Dokumentuj funkcje i komponenty
4. **Performance**: Memoize, optimize, measure
5. **Accessibility**: Nie jest opcjonalna - ARIA labels, keyboard navigation
6. **Testing**: Pisz testy dla krytycznej logiki (gdy będzie czas)
7. **Documentation**: Kod powinien być self-documenting
8. **DRY**: Don't Repeat Yourself - komponenty reusable
9. **KISS**: Keep It Simple, Stupid - nie overkilluj rozwiązań
10. **Polish First**: Komunikacja po polsku, kod po angielsku
11. **Ask for Help**: Po 2 próbach - pytaj o pomoc
12. **Modern JS**: Używaj ES6+, arrow functions, async/await, destructuring
13. **No TypeScript**: Na tym etapie bez TypeScript, skupiamy się na działaniu
14. **Error Handling**: Zawsze obsługuj błędy gracefully

---

**Wersja**: 2.0  
**Ostatnia aktualizacja**: 2025-01-18  
**Autor**: Frontend Team  
**Status**: Bez TypeScript - JavaScript ES6+

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/majkel1987)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/majkel1987)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
