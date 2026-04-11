## glowmi-net

> Premium bilingual (EN + AR with RTL) frontend for AI-powered skincare e-commerce platform targeting KSA market. Connects to backend via REST API.

# 1. GLOWMI - AI-Powered Skincare E-Commerce (Frontend)

## Overview

Premium bilingual (EN + AR with RTL) frontend for AI-powered skincare e-commerce platform targeting KSA market. Connects to backend via REST API.

## Architecture

- **This repo:** Frontend only (Next.js)
- **Backend:** Separate repo/service (REST API)
- **Communication:** REST API with JWT authentication

## Implementation Status

**✅ Implemented:**

- Homepage (hero, about, products showcase, contact, footer)
- i18n routing (en/ar), navigation with language toggle
- UI components: button, card, carousel, dropdown-menu, sheet

**🔲 To Implement:**

- Shop pages (PLP, PDP), cart, checkout flow
- User authentication UI and account pages
- AI Skin Analyzer and Chat UI
- Admin dashboard UI
- API integration layer

## Core Features

### E-Commerce UI

- Product listing and detail pages
- Cart management interface
- Checkout flow (address forms, payment selection)
- Order confirmation and history views

### AI Features UI

- **Skin Analyzer:** Quick Check + Deep Profile forms
- **Chat Assistant:** "Ask Glowmi" chat interface
- Results display with routine recommendations
- Consent modal and disclaimer display

### Admin Dashboard UI

- Orders, products, customers list views
- Analytics dashboards
- R&D Intelligence views

## Website Pages

### Public

- Homepage ✅, About, Contact, Privacy Policy, Terms, FAQ

### Shop

- Collection/Shop pages (PLP), Product Detail (PDP)
- Cart, Checkout, Order confirmation

### User Account

- Dashboard, Order history, Addresses, Profile settings

### AI

- Skin Analyzer (Quick/Deep), Chat Assistant, Results & Saved Routines

### Admin (Protected)

- Orders, Products, Customers, Analytics, R&D

## E-Commerce Flow (Frontend)

```
Browse → Add to Cart → Checkout → Address Form → Payment Selection → Confirmation → Order History
```

## API Integration Points

| Feature  | API Calls                              |
| -------- | -------------------------------------- |
| Auth     | Login, Register, Logout, Refresh token |
| Products | List, Detail, Search, Filter           |
| Cart     | Get, Add, Update, Remove items         |
| Checkout | Create order, Payment intent           |
| Orders   | List, Detail, Track                    |
| AI       | Analyze skin, Chat, Get history        |
| User     | Profile, Addresses, Preferences        |
| Admin    | Orders, Products, Customers, Analytics |

## Brand Guidelines

- Luxury aesthetic: minimal, high-end typography, generous spacing
- Mobile-first responsive, smooth transitions
- Colors: Dark green (#1a2e1a), cream/beige backgrounds (#f5f4f3, #e8e6e3)
- Fonts: Serif for headings (italic emphasis), sans-serif for body

## Required UI Elements

### Consent & Compliance

- PDPL consent modal before AI features
- Cookie consent banner
- Data deletion request UI in profile

### Disclaimers

**EN:** "Glowmi AI provides educational skincare guidance only and does not diagnose, treat, or prescribe."

**AR:** "يقدم Glowmi AI إرشادات تعليمية للعناية بالبشرة فقط ولا يقوم بالتشخيص أو العلاج أو الوصف الطبي."

## Performance Targets

- Mobile Lighthouse ≥ 85
- Page load < 3s

## Bilingual Requirements

- All UI text in translation files (no hardcoded strings)
- English (en) default, Arabic (ar) with full RTL
- All pages, forms, and messages bilingual

# 2. Code Standards & API Integration

## Code Quality Principles

- **Type Safety:** Strict TypeScript, prefer `unknown` over `any`
- **Accessibility:** Semantic HTML, ARIA attributes, keyboard handlers
- **Performance:** Efficient code, proper error handling
- **Consistency:** Follow existing patterns

## TypeScript

```typescript
// ✅ Do
export type UserRole = 'customer' | 'admin'
const users: User[] = []
import type { User } from '@/types'

// ❌ Don't
enum UserRole {
  Customer,
  Admin,
} // Use union types
const users: any = [] // Use proper types
```

- Use `as const` for literals, `T[]` for arrays
- No enums, namespaces, or non-null assertions unless necessary

## React & Next.js

- Don't define components inside components
- Use default exports for components
- Call hooks at top level only
- Include all hook dependencies
- Use `next/image`, not `<img>`
- Use `<>...</>` not `<Fragment>`
- Don't use array index as keys

## Accessibility

- Button: always include `type` attribute
- `onClick` → add keyboard handler (`onKeyUp`/`onKeyDown`)
- `onMouseOver` → add `onFocus`/`onBlur`
- Meaningful alt text (not "image", "photo")
- `lang` attribute on html element

## Error Handling

```typescript
// ✅ Good
const fetchData = async (id: string) => {
  try {
    const response = await api.get(id)
    return { success: true, data: response }
  } catch (error) {
    console.error('Failed:', error)
    return { success: false, error: error instanceof Error ? error.message : 'Unknown' }
  }
}
```

## Modern JS

- `const`/`let` only (no `var`)
- Template literals, destructuring
- `for...of` over `forEach`
- `Promise.all()` for parallel async
- Optional chaining (`?.`)

## Security

- No hardcoded secrets
- Validate/sanitize inputs
- `target="_blank"` → add `rel="noopener"`
- Store JWT in httpOnly cookies (not localStorage)

## Testing

- Descriptive test names
- Test happy path + error scenarios
- No focused tests (`fit`, `fdescribe`) in commits
- Mock API calls in tests

---

## API Integration

### API Client Setup

```typescript
// src/lib/api/client.ts
const apiClient = {
  baseURL: process.env.NEXT_PUBLIC_API_URL,
  // Include credentials for httpOnly cookies
  credentials: 'include',
}
```

### Type-Safe API Responses

```typescript
// src/types/api.ts
export type ApiResponse<T> = {
  success: boolean
  data?: T
  error?: string
}

// Validate with Zod
const userSchema = z.object({
  id: z.string(),
  email: z.string().email(),
  role: z.enum(['customer', 'admin']),
})
```

### React Query Usage

```typescript
// Use React Query for server state
const { data, isLoading, error } = useQuery({
  queryKey: ['products'],
  queryFn: () => api.products.getAll(),
})
```

### API Endpoints (Frontend calls)

| Feature  | Endpoints                                      |
| -------- | ---------------------------------------------- |
| Auth     | POST /auth/login, /auth/register, /auth/logout |
| Products | GET /products, /products/:id                   |
| Cart     | GET/POST/PUT/DELETE /cart                      |
| Orders   | GET/POST /orders                               |
| AI       | POST /ai/analyze, /ai/chat                     |
| User     | GET/PUT /user/profile, /user/addresses         |

### Error Handling

- Handle 401 → redirect to login
- Handle 403 → show forbidden message
- Handle 500 → show generic error
- Handle network errors gracefully

---

## AI Features (Frontend)

### Skin Analyzer UI

- Quick Check: text input form
- Deep Profile: multi-step form with optional photo upload
- Display loading states during API calls

### Chat Assistant UI

- Real-time chat interface
- Message history display
- Product recommendation cards

### Required UI Elements

- Consent modal before AI analysis (PDPL)
- Non-medical disclaimer display
- Loading/streaming states for AI responses

### Disclaimers (Required)

**EN:** "Glowmi AI provides educational skincare guidance only and does not diagnose, treat, or prescribe. Consult a dermatologist for medical concerns."

**AR:** "يقدم Glowmi AI إرشادات تعليمية للعناية بالبشرة فقط ولا يقوم بالتشخيص أو العلاج أو الوصف الطبي. استشر طبيب الأمراض الجلدية للمخاوف الطبية."

# 3. Tech Stack & Project Structure

## Core Stack

| Category        | Technology                                                       |
| --------------- | ---------------------------------------------------------------- |
| Framework       | Next.js 15 (App Router), React 19 (RSC), TypeScript 5.8 (strict) |
| Styling         | Tailwind CSS, shadcn/ui + Radix UI, Lucide icons, Framer Motion  |
| State/Data      | React Query (TanStack Query) for API state management            |
| Auth            | JWT tokens via REST API (stored in httpOnly cookies)             |
| i18n            | next-intl (en/ar with RTL), locale routing                       |
| Testing         | Vitest, Playwright, Testing Library, Storybook                   |
| Quality         | ESLint, Prettier, Knip, Codehawk                                 |
| Package Manager | Bun                                                              |

## API Integration

- REST API for all backend operations
- API client with interceptors for auth tokens
- Environment-based API URL configuration
- Type-safe API response handling with Zod

## Project Structure

```
├── .kiro/steering/     # AI steering docs
├── config/             # App configuration
├── public/             # Static assets
├── scripts/            # Build scripts
├── src/
│   ├── app/
│   │   ├── [locale]/           # i18n routes
│   │   │   ├── (home)/         # Homepage ✅
│   │   │   ├── (shop)/         # Shop, PLP, PDP
│   │   │   ├── (checkout)/     # Cart, checkout
│   │   │   ├── (account)/      # User account
│   │   │   ├── (ai)/           # AI features
│   │   │   ├── (pages)/        # Static pages
│   │   │   └── (admin)/        # Admin dashboard
│   ├── components/
│   │   ├── ui/         # shadcn/ui ✅
│   │   ├── shared/     # Nav, footer, headings ✅
│   │   ├── shop/, checkout/, ai/, admin/
│   ├── hooks/          # Custom hooks
│   ├── i18n/locales/   # en/, ar/
│   ├── lib/
│   │   ├── api/        # API client, endpoints
│   │   └── utils.ts    # Utilities
│   ├── styles/         # Global CSS
│   ├── types/          # TypeScript types (API responses, models)
│   ├── utils/          # Helpers
│   ├── validations/    # Zod schemas for API responses
│   ├── env.ts          # Env validation
│   └── middleware.ts   # Auth & locale middleware
```

## Conventions

### Files & Imports

- kebab-case files, PascalCase components
- `.tsx` for components, `.ts` for utilities
- `@/*` → `src/*`

### Components

- UI: `src/components/ui/` (shadcn)
- Shared: `src/components/shared/`
- Feature-specific in dedicated folders

### API Integration

- API client in `src/lib/api/`
- Type definitions in `src/types/`
- Zod validation for API responses
- React Query for server state

### i18n

- Locales in `src/i18n/locales/{en,ar}/`
- Full RTL for Arabic
- No hardcoded strings

### Styling

- Tailwind CSS classes
- CSS variables (HSL) for theming
- Brand: main-button (dark green)

## Commands

### Development

```bash
npm run dev           # Dev server
npm run build         # Production build
npm run start         # Production server
npm run type-check    # TypeScript check
```

### Quality

```bash
npm run lint          # ESLint
npm run lint:fix      # Fix lint issues
npm run format        # Prettier format
npm run knip          # Unused code check
```

## API Integration Patterns

### Query List Files (`src/api/query-list/`)

- Define TypeScript interfaces for API responses
- Create API functions using axiosClient
- Export interfaces and API object

### API Hooks (`src/api/api-hooks/`)

- Use TanStack Query's `useQuery` for GET requests
- Use TanStack Query's `useMutation` for POST/PUT/PATCH/DELETE
- Implement query keys for caching
- Handle success/error with react-hot-toast notifications
- Invalidate related queries on mutations

### Form Handling

- Use React Hook Form with Zod resolver
- Define validation schemas in `src/validations/`
- Use FormInput component for consistent form fields
- Handle loading states during form submission

# 4. API Architecture Pattern

## Overview

This project uses a **two-layer architecture** for type-safe API integration. The pattern separates pure API definitions from React-specific hooks, making the codebase maintainable, testable, and reusable.

**Tech Stack:** TanStack Query (React Query) + TypeScript + Axios

**Key Principle:** Separation of concerns - API logic is independent of React, while hooks provide React-specific functionality.

---

## Folder Structure

```
src/
├── api/
│   ├── query-list/          # Layer 1: Pure API definitions
│   │   ├── products.ts      # Product API endpoints
│   │   ├── auth.ts          # Authentication endpoints
│   │   ├── cart.ts          # Cart endpoints
│   │   └── orders.ts        # Order endpoints
│   │
│   └── api-hooks/           # Layer 2: React Query hooks
│       ├── useProducts.ts   # Product hooks (useProducts, useCreateProduct, etc.)
│       ├── useAuth.ts       # Auth hooks (useLogin, useRegister, etc.)
│       ├── useCart.ts       # Cart hooks (useCart, useAddToCart, etc.)
│       └── useOrders.ts     # Order hooks (useOrders, useCreateOrder, etc.)
│
└── lib/
    └── axios.ts             # Axios client configuration
```

---

## Layer 1: Query List (Pure API)

**Location:** `src/api/query-list/`

**Purpose:** Define API endpoints and TypeScript types without any React dependencies. These are pure functions that can be used anywhere (components, server actions, tests).

**Structure:**

1. TypeScript interfaces for data types
2. API object with endpoint methods
3. No React imports, no hooks, no side effects

**Example:**

```typescript
// src/api/query-list/products.ts
import { axiosClient } from '@/lib/axios'

// 1. Define TypeScript interfaces
export interface Product {
  id: number
  name: string
  price: number
}

export interface ProductFilters {
  search?: string
  category?: string
}

// 2. Create API object
export const productsApi = {
  getAll: (filters?: ProductFilters) =>
    axiosClient.get<Product[]>('/products/', { params: filters }),

  getById: (id: number) => axiosClient.get<Product>(`/products/${id}/`),

  create: (data: Omit<Product, 'id'>) => axiosClient.post<Product>('/products/', data),
}
```

---

## Layer 2: API Hooks (React Query)

**Location:** `src/api/api-hooks/`

**Purpose:** Provide React hooks with caching, loading states, error handling, and automatic refetching. These hooks use the API functions from Layer 1.

**Structure:**

1. Query keys constant (for cache management)
2. Query hooks for GET requests (useQuery)
3. Mutation hooks for POST/PATCH/DELETE (useMutation)
4. Error handling with toast notifications
5. Cache invalidation after mutations

**Example:**

```typescript
// src/api/api-hooks/useProducts.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query'
import { toast } from 'react-hot-toast'
import { productsApi } from '../query-list/products'

// 1. Query keys for cache management
const PRODUCT_KEYS = {
  all: (filters?) => ['products', filters] as const,
  detail: (id: number) => ['products', id] as const,
}

// 2. Query hook (GET)
export const useProducts = (filters?) => {
  return useQuery({
    queryKey: PRODUCT_KEYS.all(filters),
    queryFn: () => productsApi.getAll(filters),
    select: (response) => response.data,
  })
}

// 3. Mutation hook (POST)
export const useCreateProduct = () => {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: productsApi.create,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['products'] })
      toast.success('Product created!')
    },
    onError: () => toast.error('Failed to create product'),
  })
}
```

---

## Usage in Components

### Fetching Data (Query)

```typescript
import { useProducts } from '@/api/api-hooks/useProducts'

function ProductList() {
  const { data, isLoading, error } = useProducts({ category: 'skincare' })

  if (isLoading) return <Spinner />
  if (error) return <ErrorMessage />

  return (
    <div>
      {data?.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  )
}
```

### Mutating Data (Create/Update/Delete)

```typescript
import { useCreateProduct } from '@/api/api-hooks/useProducts'

function CreateProductForm() {
  const { mutate, isPending } = useCreateProduct()

  const handleSubmit = (formData) => {
    mutate(formData)
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="name" />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Creating...' : 'Create Product'}
      </button>
    </form>
  )
}
```

---

## Common Patterns

### File Uploads (FormData)

```typescript
// Query List
export const uploadApi = {
  uploadImage: (file: File) => {
    const formData = new FormData()
    formData.append('image', file)
    return axiosClient.post('/upload/', formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    })
  },
}

// Hook
export const useUploadImage = () => {
  return useMutation({
    mutationFn: uploadApi.uploadImage,
    onSuccess: () => toast.success('Image uploaded!'),
  })
}
```

### Conditional Queries (Only fetch when needed)

```typescript
export const useProduct = (id?: number) => {
  return useQuery({
    queryKey: ['products', id],
    queryFn: () => productsApi.getById(id!),
    enabled: !!id, // Only fetch when ID exists
  })
}
```

### Pagination

```typescript
export const useProducts = (page: number) => {
  return useQuery({
    queryKey: ['products', { page }],
    queryFn: () => productsApi.getAll({ page }),
    placeholderData: (prev) => prev, // Keep old data while fetching new
  })
}
```

---

## Rules & Best Practices

### Layer 1: Query List ✅ DO / ❌ DON'T

✅ **DO:**

- Export all TypeScript interfaces
- Use descriptive, consistent naming
- Type all API responses
- Keep functions pure (no side effects)
- Use `URLSearchParams` for query strings

❌ **DON'T:**

- Import React or React Query
- Add business logic or state management
- Handle errors (let hooks handle them)
- Use hooks or React-specific code

### Layer 2: API Hooks ✅ DO / ❌ DON'T

✅ **DO:**

- Define query keys at the top as constants
- Include filters/params in query keys
- Use `select` to transform response data
- Invalidate queries after mutations
- Show toast notifications for user feedback
- Type error responses properly

❌ **DON'T:**

- Hardcode query keys inline
- Forget to invalidate cache after mutations
- Skip error handling
- Mix API logic with component logic

---

## Implementation Workflow

When adding a new API feature (e.g., "Orders"):

1. **Create Query List** (`src/api/query-list/orders.ts`)
   - Define TypeScript interfaces (Order, OrderFilters, CreateOrderData)
   - Create API object with methods (getAll, getById, create, update, delete)

2. **Create API Hooks** (`src/api/api-hooks/useOrders.ts`)
   - Define query keys constant
   - Create query hooks for GET requests
   - Create mutation hooks for POST/PATCH/DELETE
   - Add error handling with toast
   - Implement cache invalidation

3. **Use in Components**
   - Import hooks from `@/api/api-hooks/useOrders`
   - Use query hooks for fetching data
   - Use mutation hooks for creating/updating/deleting

---

## Benefits

- **Type Safety:** Full TypeScript support with autocomplete
- **Caching:** Automatic caching and deduplication with React Query
- **Reusability:** API functions work in any context (components, server, tests)
- **Testability:** Easy to mock each layer independently
- **Maintainability:** Clear separation of concerns
- **Developer Experience:** Predictable patterns, less boilerplate
- **Performance:** Automatic request deduplication and background refetching

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yeasin2002)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yeasin2002)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
