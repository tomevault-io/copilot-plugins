## vue3-senior-engineering-skill

> |


# Vue 3 Senior Engineering Skill

A comprehensive skill for generating production-grade Vue 3 code following architectural best practices, team conventions, and senior-level engineering standards.

---

## Table of Contents

1. [Core Principles](#core-principles)
2. [When to Use This Skill](#when-to-use-this-skill)
3. [Workflow: Planning Before Code](#workflow-planning-before-code)
4. [Component Architecture](#component-architecture)
5. [State Management & Pinia](#state-management--pinia)
6. [Composables & Reusable Logic](#composables--reusable-logic)
7. [Testing Strategy](#testing-strategy)
8. [HTTP & API Integration](#http--api-integration)
9. [Styling with Tailwind CSS](#styling-with-tailwind-css)
10. [Nuxt-Specific Patterns](#nuxt-specific-patterns)
11. [Code Quality Standards](#code-quality-standards)
12. [Performance & Optimization](#performance--optimization)
13. [Architecture Decision Records](#architecture-decision-records)
14. [Appendix: Quick Reference](#appendix-quick-reference)

---

## Core Principles

This skill enforces a unified set of architectural and coding standards designed for scalability, maintainability, and long-term team success:

1. **Production-Grade First**: All code is optimized, structured, and test-ready from day one
2. **Composition API Always**: Vue 3 Composition API is the standard; Options API only for legacy code
3. **Separation of Concerns**: Clear boundaries between UI logic, business logic, and data access
4. **Type Safety Enforced**: Strict TypeScript mode, no `any` types, explicit return types everywhere
5. **Pragmatism Over Dogma**: Architecture serves the team and users, not rules for their own sake
6. **Senior-Level Collaboration**: Ask clarifying questions, offer alternatives with rationale, guide decisions
7. **Performance Conscious**: Minimize re-renders, optimize reactivity, reduce bundle bloat
8. **Testability Built-In**: Design for testing; include factory functions and test patterns by default
9. **Documentation & Clarity**: Code is readable, maintainable, and well-documented
10. **Consistency Across Teams**: Enforce naming conventions, file organization, and patterns uniformly

---

## When to Use This Skill

### Explicit Triggers (Always Use)

- **Writing new Vue 3 components** (feature components, base components, containers)
- **Refactoring existing Vue code** (Options API → Composition API, improving structure)
- **Creating or updating composables** (reusable logic, hooks)
- **Building/updating Pinia stores** (state management, actions, getters)
- **Creating Vue plugins** (custom functionality, global features)
- **Performance optimization** (reducing re-renders, optimizing reactivity, code splitting)
- **Frontend architecture discussions** (folder structure, data flow, patterns)
- **Debugging Vue-specific patterns** (reactivity issues, lifecycle, watchers)
- **Converting Options API to Composition API** (modernizing legacy code)
- **Optimizing rendering, reactivity, or state** handling
- **Code reviews of Vue code** (enforcing standards, suggesting improvements)
- **Building HTTP clients and API services** (fetch integration, request/response transformation)
- **Setting up Nuxt applications** (SSR, auto-imports, middleware, server boundaries)
- **Writing validation schemas** (Zod integration, type safety)
- **Creating utility functions** (one function per file, global imports)
- **Configuring Tailwind CSS** (design systems, utility composition)

### Input Scenarios

The skill accepts:
- Existing `.vue` files (components)
- `.ts` / `.js` files (composables, utilities, services)
- Pinia store files
- Plugin files
- Feature requirements (description of desired behavior)
- UI requirements (design specs, layout needs)
- Refactor requests (code smell, architectural issues)
- Bug reports (reactivity, performance, behavior issues)
- Architecture discussions (structure, patterns, data flow)
- Performance concerns (slow renders, bundle size, memory)
- Code snippets and partial implementations
- Configuration files (Tailwind, Vite, Nuxt)

---

## Workflow: Planning Before Code

### Step 1: Clarification & Planning (Always Start Here)

**Never jump straight to code generation.** Always plan first.

Ask clarifying questions to understand:
1. **What is the feature or task?** (Clear description, context, requirements)
2. **What's the current state?** (Existing code, architecture, constraints)
3. **What problem are we solving?** (User impact, business value, technical goal)
4. **Are there existing patterns?** (In the codebase, team standards)
5. **Scale & complexity:** (Single component, feature, refactor, architecture?)
6. **Constraints:** (Performance, compatibility, accessibility, security)
7. **Integration points:** (API, stores, other components, routing)

**Output: Clear Plan Before Code**

Once you understand the context, propose a plan (not code):

```markdown
## Plan

### Overview
[Clear description of what we're building]

### Structure
- [Component hierarchy or file organization]
- [State management approach]
- [API integration points]

### Key Decisions
- [Architecture patterns]
- [Composable/store breakdown]
- [Performance considerations]

### Implementation Steps
1. [Create base structure]
2. [Implement core logic]
3. [Add tests]
4. [Optimize]

### Questions Before We Proceed
- [Any unresolved decisions?]
```

Ask: **"Does this plan align with what you need? Any changes before we proceed with implementation?"**

---

## Component Architecture

### Component Types & Responsibilities

**Three primary types:**

#### 1. Base/Atomic Components

**Purpose:** Foundational, reusable building blocks (no business logic)

**Examples:** Button, Input, Card, Badge, Modal, Alert

**Characteristics:**
- Single, narrow responsibility
- Zero business logic (purely presentational)
- Fully configurable via props
- Handle all states: default, loading, error, disabled, empty
- WCAG 2.1 AA accessibility built-in
- Full TypeScript types, no `any`
- Comprehensive inline documentation

See detailed example in `references/examples/BaseComponent.vue`

#### 2. Feature/Container Components

**Purpose:** Orchestrate multiple base components + business logic

**Examples:** ProductList (with API call), UserProfile (with store), ArticleDetail

**Characteristics:**
- Connect to stores (Pinia) or API services
- Handle data fetching and transformations
- Manage complex state transitions
- Coordinate child components
- Should remain **< 500 lines**; split if larger
- Own their error/loading/empty state handling


### Component Design Checklist

Before shipping any component, verify:

- [ ] Single, clear responsibility
- [ ] Props are fully typed (TypeScript strict mode)
- [ ] Works independently without external context
- [ ] Events emitted for all significant interactions
- [ ] Internal state limited to UI concerns
- [ ] Handles loading, error, and empty states
- [ ] Accessibility: WCAG 2.1 AA minimum
- [ ] Performance: <500 lines, optimized re-renders
- [ ] Testable: clear inputs/outputs, mockable dependencies
- [ ] Documented: JSDoc/comments for public API
- [ ] Works with design system and Tailwind CSS

### Component File Organization

```
src/components/
├── base/                  # Atomic components
│   ├── Button.vue
│   ├── Input.vue
│   ├── Card.vue
│   ├── Badge.vue
│   ├── Modal.vue
│   └── __tests__/
│       └── Button.spec.ts
├── common/                # Cross-feature components
│   ├── Header.vue
│   ├── Sidebar.vue
│   ├── Footer.vue
│   └── __tests__/
├── features/              # Feature-specific components
│   ├── products/
│   │   ├── ProductList.vue
│   │   ├── ProductCard.vue
│   │   ├── ProductDetail.vue
│   │   └── __tests__/
│   └── users/
│       ├── UserProfile.vue
│       ├── UserAvatar.vue
│       └── __tests__/
```

---

## State Management & Pinia

### Store Organization

**Naming Convention:** `kebab-case.ts`

**One store per feature/domain:**

```
src/stores/
├── auth-store.ts          # Authentication state
├── user-store.ts          # User profile & settings
├── product-store.ts       # Product data
├── cart-store.ts          # Shopping cart
└── __tests__/
    └── product-store.spec.ts
```

### Store Structure Template

```typescript
// src/stores/product-store.ts
import { defineStore } from 'pinia';
import { ref, computed } from 'vue';
import { productService } from '@/services/product-service';
import type { Product } from '@/types';

export const useProductStore = defineStore('product', () => {
  // ============================================================================
  // State
  // ============================================================================
  const products = ref<Product[]>([]);
  const selectedProductId = ref<string | null>(null);
  const isLoading = ref(false);
  const error = ref<string | null>(null);

  // ============================================================================
  // Computed (Getters)
  // ============================================================================
  const selectedProduct = computed(() =>
    products.value.find((p) => p.id === selectedProductId.value)
  );

  const productCount = computed(() => products.value.length);

  // Never store derived data—compute it instead
  const sortedProducts = computed(() =>
    [...products.value].sort((a, b) => a.name.localeCompare(b.name))
  );

  // ============================================================================
  // Actions (Async Operations)
  // ============================================================================
  const fetchProducts = async () => {
    isLoading.value = true;
    error.value = null;

    try {
      products.value = await productService.getProducts();
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Unknown error';
      throw err;
    } finally {
      isLoading.value = false;
    }
  };

  const selectProduct = (id: string) => {
    selectedProductId.value = id;
  };

  const addProduct = async (product: Omit<Product, 'id'>) => {
    try {
      const newProduct = await productService.createProduct(product);
      products.value.push(newProduct);
      return newProduct;
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Failed to add product';
      throw err;
    }
  };

  const clearError = () => {
    error.value = null;
  };

  // ============================================================================
  // Return
  // ============================================================================
  return {
    // State
    products,
    selectedProductId,
    isLoading,
    error,
    // Computed
    selectedProduct,
    productCount,
    sortedProducts,
    // Actions
    fetchProducts,
    selectProduct,
    addProduct,
    clearError,
  };
});
```

### Store Best Practices

1. **Keep state flat** (avoid deeply nested objects)
2. **Use getters for derived data** (never store computed values)
3. **Async operations go in actions** (not in watchers)
4. **Clear error states** after user acknowledges
5. **Document complex mutations** with inline comments
6. **One store per feature** (not per component)
7. **State persistence intentional** (document localStorage usage)
8. **Type-safe getters** (explicit return types)

### State Categories

| Category | Location | Persistence | Example |
|----------|----------|-------------|---------|
| **UI State** | Component `ref` | Session | Dropdown open, modal visible, form input |
| **App State** | Pinia Store | Often persisted | User auth, preferences, theme |
| **Server State** | Pinia Store (cached) | Cached | API data, lists, search results |
| **Derived State** | Computed property | Never stored | Filtered lists, formatted dates |

---

## Composables & Reusable Logic

### Composable Design

**Naming Convention:** `use*` prefix, camelCase

**Location:** `src/composables/`

**Principle:** One responsibility, focused purpose

### Composable Structure Template

```typescript
// src/composables/useFetchProducts.ts
import { ref, computed } from 'vue';
import { productService } from '@/services/product-service';
import type { Product } from '@/types';

/**
 * Composable for fetching and managing product data
 * @returns Object with data, loading state, error, and refetch method
 */
export const useFetchProducts = () => {
  // ============================================================================
  // State
  // ============================================================================
  const data = ref<Product[]>([]);
  const isLoading = ref(false);
  const error = ref<string | null>(null);

  // ============================================================================
  // Computed
  // ============================================================================
  const hasError = computed(() => error.value !== null);
  const isEmpty = computed(() => data.value.length === 0);

  // ============================================================================
  // Methods
  // ============================================================================
  const fetch = async () => {
    isLoading.value = true;
    error.value = null;

    try {
      data.value = await productService.getProducts();
    } catch (err) {
      error.value = err instanceof Error ? err.message : 'Failed to fetch';
    } finally {
      isLoading.value = false;
    }
  };

  const reset = () => {
    data.value = [];
    error.value = null;
    isLoading.value = false;
  };

  // ============================================================================
  // Return (organized by type: state, computed, methods)
  // ============================================================================
  return {
    // State
    data,
    isLoading,
    error,
    // Computed
    hasError,
    isEmpty,
    // Methods
    fetch,
    reset,
  };
};
```

### Common Composable Patterns

#### useAuth
```typescript
// Manages authentication state, login, logout, user info
const { user, isAuthenticated, login, logout, register, refresh } = useAuth();
```

#### useFetch (Data Loading)
```typescript
// Wraps API call with loading/error handling
const { data, isLoading, error, refetch } = useFetch(() => api.get('/users'));
```

#### usePagination
```typescript
// Manages pagination state
const { currentPage, pageSize, total, hasNextPage, goToPage } = usePagination(items, 10);
```

#### useForm
```typescript
// Manages form state, validation, submission
const { form, errors, isDirty, submit, reset } = useForm(schema, onSubmit);
```

### Composable Best Practices

1. **Single responsibility** (do one thing well)
2. **Clear return object** (organize by type: state, computed, methods)
3. **Type-safe** (explicit return types, no `any`)
4. **No side effects in setup** (side effects go in watchers or lifecycle hooks)
5. **Avoid creating refs in loops** (creates unnecessary memory)
6. **Document publicly exported functions** (JSDoc comments)
7. **Test in isolation** (mock external dependencies)

---

## Testing Strategy

### Test Stack: Vitest + Vue Test Utils

**File Naming:** `{name}.spec.ts`

**Locations:**
- Component specs: `src/components/__tests__/{feature}/`
- Composable specs: `src/composables/__tests__/`
- Store specs: `src/stores/__tests__/`
- Utility specs: `src/utils/__tests__/`

### Test File Structure

```typescript
// src/components/__tests__/features/ProductCard.spec.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { mount } from '@vue/test-utils';
import ProductCard from '@/components/features/ProductCard.vue';
import type { Product } from '@/types';

// ============================================================================
// Factories (Create test data)
// ============================================================================
const createProductMock = (overrides?: Partial<Product>): Product => ({
  id: '1',
  name: 'Test Product',
  price: 99.99,
  description: 'A test product',
  ...overrides,
});

// ============================================================================
// Test Suites
// ============================================================================
describe('ProductCard', () => {
  // ============================================================================
  // Rendering
  // ============================================================================
  describe('rendering', () => {
    it('renders product name and price', () => {
      const product = createProductMock();
      const wrapper = mount(ProductCard, { props: { product } });

      expect(wrapper.text()).toContain(product.name);
      expect(wrapper.text()).toContain(product.price);
    });

    it('renders description when provided', () => {
      const product = createProductMock({ description: 'Custom description' });
      const wrapper = mount(ProductCard, { props: { product } });

      expect(wrapper.text()).toContain('Custom description');
    });
  });

  // ============================================================================
  // Interactions
  // ============================================================================
  describe('interactions', () => {
    it('emits select event when clicked', async () => {
      const product = createProductMock();
      const wrapper = mount(ProductCard, { props: { product } });

      await wrapper.find('[data-testid="product-card"]').trigger('click');

      expect(wrapper.emitted('select')).toHaveLength(1);
      expect(wrapper.emitted('select')?.[0]).toEqual([product.id]);
    });
  });

  // ============================================================================
  // Edge Cases
  // ============================================================================
  describe('edge cases', () => {
    it('handles missing description gracefully', () => {
      const product = createProductMock({ description: undefined });
      const wrapper = mount(ProductCard, { props: { product } });

      expect(wrapper.find('[data-testid="description"]').exists()).toBe(false);
    });
  });
});
```

### Composable Testing

```typescript
// src/composables/__tests__/useFetchProducts.spec.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { useFetchProducts } from '@/composables/useFetchProducts';
import * as productService from '@/services/product-service';

vi.mock('@/services/product-service');

describe('useFetchProducts', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('fetches products on demand', async () => {
    const mockProducts = [
      { id: '1', name: 'Product 1' },
      { id: '2', name: 'Product 2' },
    ];

    vi.spyOn(productService, 'getProducts').mockResolvedValue(mockProducts);

    const { data, isLoading, fetch } = useFetchProducts();

    expect(isLoading.value).toBe(false);

    await fetch();

    expect(isLoading.value).toBe(false);
    expect(data.value).toEqual(mockProducts);
  });

  it('handles fetch errors', async () => {
    const error = new Error('API error');
    vi.spyOn(productService, 'getProducts').mockRejectedValue(error);

    const { error: fetchError, fetch } = useFetchProducts();

    await fetch();

    expect(fetchError.value).toBe('API error');
  });
});
```

### Store Testing

```typescript
// src/stores/__tests__/product-store.spec.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { setActivePinia, createPinia } from 'pinia';
import { useProductStore } from '@/stores/product-store';
import * as productService from '@/services/product-service';

vi.mock('@/services/product-service');

describe('Product Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia());
    vi.clearAllMocks();
  });

  it('fetches and stores products', async () => {
    const mockProducts = [{ id: '1', name: 'Product' }];
    vi.spyOn(productService, 'getProducts').mockResolvedValue(mockProducts);

    const store = useProductStore();
    await store.fetchProducts();

    expect(store.products).toEqual(mockProducts);
  });
});
```

### Testing Best Practices

1. **Test behavior, not implementation** (focus on inputs/outputs)
2. **Use factory functions** for test data creation
3. **Mock external dependencies** (services, API calls)
4. **Test all states** (loading, error, success, empty)
5. **Test edge cases** (null values, empty arrays, etc.)
6. **Keep tests focused** (one test, one behavior)
7. **Use descriptive test names** (what should happen)
8. **Organize tests by concern** (rendering, interactions, edge cases)

### Coverage Target: 80%

---

## HTTP & API Integration

### Setup: Fetch API Client

**Location:** `src/services/http.ts`

```typescript
// src/services/http.ts
interface FetchConfig extends RequestInit {
  baseURL?: string;
  params?: Record<string, any>;
  timeout?: number;
}

interface ApiResponse<T = unknown> {
  data: T;
  status: number;
  headers: Headers;
}

/**
 * Create a configured HTTP client
 */
const createHttpClient = (baseURL: string) => {
  const buildUrl = (path: string, params?: Record<string, any>): string => {
    const url = new URL(path, baseURL);
    if (params) {
      Object.entries(params).forEach(([key, value]) => {
        url.searchParams.append(key, String(value));
      });
    }
    return url.toString();
  };

  const request = async <T>(
    path: string,
    method: string,
    body?: any,
    config?: FetchConfig
  ): Promise<ApiResponse<T>> => {
    const url = buildUrl(path, config?.params);
    
    const response = await fetch(url, {
      method,
      headers: {
        'Content-Type': 'application/json',
        ...config?.headers,
      },
      body: body ? JSON.stringify(transformRequest(body)) : undefined,
      ...config,
    });

    const data = await response.json();

    if (!response.ok) {
      throw new ApiError(data.message || 'Request failed', response.status, data);
    }

    return {
      data: transformResponse<T>(data),
      status: response.status,
      headers: response.headers,
    };
  };

  return {
    get: <T>(path: string, config?: FetchConfig) => request<T>(path, 'GET', undefined, config),
    post: <T>(path: string, body?: any, config?: FetchConfig) => request<T>(path, 'POST', body, config),
    put: <T>(path: string, body?: any, config?: FetchConfig) => request<T>(path, 'PUT', body, config),
    delete: <T>(path: string, config?: FetchConfig) => request<T>(path, 'DELETE', undefined, config),
  };
};

export const http = createHttpClient(import.meta.env.VITE_API_BASE_URL);
```

### Request/Response Transformation

```typescript
// src/services/transformers.ts
import { toSnakeCase, toCamelCase } from '@/utils';

/**
 * Transform request data: camelCase → snake_case
 */
export const transformRequest = (data: Record<string, any>): Record<string, any> => {
  if (Array.isArray(data)) {
    return data.map(transformRequest);
  }
  
  if (data && typeof data === 'object') {
    return Object.entries(data).reduce((acc, [key, value]) => {
      acc[toSnakeCase(key)] = value;
      return acc;
    }, {});
  }
  
  return data;
};

/**
 * Transform response data: snake_case → camelCase
 */
export const transformResponse = <T>(data: any): T => {
  if (Array.isArray(data)) {
    return data.map(transformResponse) as T;
  }
  
  if (data && typeof data === 'object') {
    return Object.entries(data).reduce((acc, [key, value]) => {
      acc[toCamelCase(key)] = transformResponse(value);
      return acc;
    }, {}) as T;
  }
  
  return data as T;
};
```

### Service Layer Pattern

```typescript
// src/services/product-service.ts
import { http } from './http';
import type { Product } from '@/types';

export const productService = {
  async getProducts(): Promise<Product[]> {
    const response = await http.get<Product[]>('/products');
    return response.data;
  },

  async getProduct(id: string): Promise<Product> {
    const response = await http.get<Product>(`/products/${id}`);
    return response.data;
  },

  async createProduct(data: Omit<Product, 'id'>): Promise<Product> {
    const response = await http.post<Product>('/products', data);
    return response.data;
  },

  async updateProduct(id: string, data: Partial<Product>): Promise<Product> {
    const response = await http.put<Product>(`/products/${id}`, data);
    return response.data;
  },

  async deleteProduct(id: string): Promise<void> {
    await http.delete(`/products/${id}`);
  },
};
```

### Plugin Injection

```typescript
// src/plugins/http-plugin.ts
import { App } from 'vue';
import { http } from '@/services/http';

declare module 'vue' {
  interface ComponentCustomProperties {
    $http: typeof http;
  }
}

export const httpPlugin = (app: App) => {
  app.provide('$http', http);
  app.config.globalProperties.$http = http;
};

// main.ts
import { httpPlugin } from '@/plugins/http-plugin';
app.use(httpPlugin);
```

### Usage in Composables/Components

```typescript
// Via inject (Composition API, preferred)
import { inject } from 'vue';
const http = inject('$http');

// Or direct import (also acceptable)
import { http } from '@/services/http';
```

---

## Styling with Tailwind CSS

### Tailwind CSS v4

**Principle:** Utility-first, composition-based, no component-level scoping

### Global Configuration

```css
/* src/styles/globals.css */
@import "tailwindcss";

/* ============================================================================
   CSS Variables (Design Tokens)
   ============================================================================ */
:root {
  --color-primary: theme(colors.blue.600);
  --color-primary-dark: theme(colors.blue.700);
  --color-danger: theme(colors.red.600);
  --spacing-xs: theme(spacing.2);
  --spacing-sm: theme(spacing.4);
}

/* ============================================================================
   Reusable Class Compositions (Use Sparingly)
   ============================================================================ */
@layer components {
  /* Button styles (reusable across base component) */
  .btn-primary {
    @apply inline-flex items-center justify-center
           px-4 py-2 rounded font-medium
           bg-blue-600 text-white
           hover:bg-blue-700
           active:bg-blue-800
           disabled:opacity-50 disabled:cursor-not-allowed;
  }

  .btn-secondary {
    @apply inline-flex items-center justify-center
           px-4 py-2 rounded font-medium
           border-2 border-gray-300 text-gray-700
           hover:bg-gray-100
           disabled:opacity-50;
  }

  /* Form inputs */
  .input-base {
    @apply w-full px-3 py-2
           border border-gray-300 rounded
           focus:outline-none focus:ring-2 focus:ring-blue-500;
  }

  /* Layout utilities */
  .container-padding {
    @apply px-4 py-6 sm:px-6 lg:px-8;
  }
}

/* ============================================================================
   Global Utilities (Minimize These)
   ============================================================================ */
body {
  @apply bg-white text-gray-900;
}
```

### Component Usage Pattern

**Avoid scoped styles. Compose utilities directly in templates:**

```vue
<!-- ✗ Avoid: Scoped CSS -->
<script setup>
// Component logic
</script>

<template>
  <button class="my-button">Click me</button>
</template>

<style scoped>
.my-button {
  @apply px-4 py-2 bg-blue-600;
}
</style>

<!-- ✓ Prefer: Utility Composition -->
<template>
  <button class="inline-flex items-center justify-center
                   px-4 py-2 rounded font-medium
                   bg-blue-600 text-white
                   hover:bg-blue-700
                   disabled:opacity-50">
    Click me
  </button>
</template>
```

### Responsive Design

```vue
<template>
  <!-- Stack on mobile, grid on tablet+, flex on desktop -->
  <div class="flex flex-col gap-4 md:grid md:grid-cols-2 lg:flex lg:flex-row">
    <div class="flex-1">Card 1</div>
    <div class="flex-1">Card 2</div>
  </div>
</template>
```

### Dark Mode (if supported)

```css
@media (prefers-color-scheme: dark) {
  :root {
    --color-primary: theme(colors.blue.400);
  }
}
```

### Tailwind Best Practices

1. **Utility composition over custom CSS** (keep styles in templates)
2. **Design tokens via CSS variables** (theme consistency)
3. **No component-level scoping** (prefer global utilities)
4. **Responsive-first** (mobile → tablet → desktop)
5. **Semantic class names** (`.btn-primary`, not `.blue-button`)
6. **Reuse only high-frequency patterns** (buttons, inputs, cards)
7. **Keep globals.css minimal** (< 100 lines of custom CSS)

---

## Nuxt-Specific Patterns

### Project Structure

```
nuxt-app/
├── app.vue                # Root component
├── nuxt.config.ts         # Nuxt configuration
├── pages/                 # Auto-routed pages
│   ├── index.vue
│   ├── products.vue
│   └── products/
│       └── [id].vue       # Dynamic routes
├── components/            # Auto-registered
├── composables/           # Auto-imported
├── stores/                # Pinia stores
├── utils/                 # Auto-imported (optional)
├── middleware/            # Route guards
├── plugins/               # Plugins
├── public/                # Static assets
└── server/                # Server routes
    ├── api/
    ├── middleware/
    └── utils/
```

### Auto-Imports Configuration

```typescript
// nuxt.config.ts
export default defineNuxtConfig({
  imports: {
    dirs: ['./composables', './utils'],
  },
  components: {
    dirs: ['./components/base', './components/features', './components/common'],
  },
});
```

### Server/Client Boundaries

```typescript
// composables/useServerData.server.ts
// Runs only on server
export const useServerData = async () => {
  const data = await fetch('http://internal-api/data');
  return data;
};

// composables/useClientData.ts
// Runs on client (browser)
export const useClientData = () => {
  const data = ref(null);
  onMounted(() => {
    // Browser-only logic
  });
  return { data };
};
```

### Middleware (Route Guards)

```typescript
// middleware/auth.ts
export default defineRouteMiddleware((to, from) => {
  const authStore = useAuthStore();
  
  if (!authStore.isAuthenticated) {
    return navigateTo('/login');
  }
});

// pages/admin.vue
definePageMeta({
  middleware: 'auth',
});
```

### Server Routes & API

```typescript
// server/api/products.get.ts
export default defineEventHandler(async (event) => {
  return await getProducts();
});

// Usage in component
const { data: products } = await useFetch('/api/products');
```

### $fetch (Server-Safe HTTP)

```typescript
// Works in both SSR and client
const data = await $fetch('/api/users');

// With transformations
const users = await $fetch('/api/users', {
  transform: (response) => response.data,
});
```

### useAsyncData (SSR-Safe Data Fetching)

```typescript
// pages/products.vue
const { data, pending, error, refresh } = await useAsyncData(
  'products',
  () => $fetch('/api/products')
);
```

### useRouter & useRoute

```typescript
// Navigation
const router = useRouter();
router.push('/products');

// Current route
const route = useRoute();
const productId = route.params.id;
```

---

## Code Quality Standards

### TypeScript

- **Mode:** Strict (`"strict": true` in tsconfig.json)
- **No `any` types** (exception: truly unavoidable legacy code)
- **Explicit return types** on all functions
- **Props & emits fully typed** (no partial typing)
- **Use discriminated unions** for complex types
- **Type predicates** for runtime type guards

```typescript
// ✓ Good
const formatDate = (date: Date): string => {
  return date.toISOString();
};

interface User {
  id: string;
  name: string;
  role: 'admin' | 'user' | 'guest';
}

// ✗ Avoid
const formatDate = (date: any) => {
  return date.toISOString();
};
```

### Prettier Formatting

- **Enforce Prettier** (not ESLint for formatting)
- **No semicolons**
- **Print width:** 80-100 characters
- **Trailing comma:** ES5 (preserve compatibility)
- **Single quotes** for strings

**.prettierrc Configuration:**
```json
{
  "semi": false,
  "singleQuote": true,
  "printWidth": 100,
  "trailingComma": "es5",
  "arrowParens": "always"
}
```

### Code Comments

**Comment "why", not "what":**

```typescript
// ✗ Avoid (Obvious from code)
const isActive = true; // Set isActive to true

// ✓ Good (Explains reasoning)
// Cache products for 5 minutes to reduce API calls
const cacheTimeMs = 5 * 60 * 1000;

// Fetch products on mount only if not cached
if (!cache.has('products') || cache.isStale('products', cacheTimeMs)) {
  await fetchProducts();
}
```

### Documentation

**JSDoc for public APIs:**

```typescript
/**
 * Format a date to human-readable format
 * @param date - Date to format
 * @param locale - Optional locale (defaults to 'en-US')
 * @returns Formatted date string
 * @example
 * formatDate(new Date(), 'en-US') // "Monday, January 1, 2024"
 */
export const formatDate = (date: Date, locale = 'en-US'): string => {
  return new Intl.DateTimeFormat(locale, {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(date);
};
```

### Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| **Components** | PascalCase | `ProductCard.vue`, `UserProfile.vue` |
| **Composables** | camelCase, `use*` prefix | `useAuth.ts`, `usePagination.ts` |
| **Stores** | kebab-case | `auth-store.ts`, `user-store.ts` |
| **Services** | camelCase | `productService.ts`, `authService.ts` |
| **Utilities** | camelCase | `formatDate.ts`, `validateEmail.ts` |
| **Types/Interfaces** | PascalCase | `User`, `Product`, `ApiResponse` |
| **Constants** | UPPER_SNAKE_CASE | `MAX_RETRIES`, `API_TIMEOUT` |
| **Private functions** | `_` prefix (optional) | `_processData`, `_validateInput` |

### Component Size Guidelines

- **< 200 lines:** Ideal
- **200-500 lines:** Acceptable; consider refactoring
- **> 500 lines:** Refactor into smaller units

---

## Performance & Optimization

### Component-Level Optimizations

#### 1. Rendering Performance
```typescript
// ✓ Use v-show for frequent toggling (keeps in DOM)
<div v-show="isVisible">Content</div>

// ✓ Use v-if for rare visibility (removes from DOM)
<div v-if="showModal">Modal</div>

// ✗ Avoid unnecessary re-renders
// Use computed properties instead of watchers
const displayPrice = computed(() => price.value.toFixed(2));
```

#### 2. Efficient Reactivity
```typescript
// ✓ Prefer computed over watchers
const doubledCount = computed(() => count.value * 2);

// ✗ Only use watchers for side effects
watch(() => count.value, (newValue) => {
  logger.log(`Count changed to ${newValue}`);
});

// ✓ Use shallow refs for large objects
const config = shallowRef({ /* large object */ });

// Avoid creating refs in loops
const items = computed(() =>
  data.value.map(item => ({ ...item, id: item.id }))
);
```

#### 3. Code Splitting
```typescript
// ✓ Lazy-load routes
const ProductPage = defineAsyncComponent(() =>
  import('@/pages/ProductPage.vue')
);

// ✓ Dynamic imports for heavy utilities
const heavyLib = await import('@/lib/heavy-library');

// ✓ Use Nuxt lazy components
const Chart = defineAsyncComponent({
  loader: () => import('@/components/Chart.vue'),
  loadingComponent: LoadingSpinner,
  delay: 200,
});
```

#### 4. Image Optimization
```html
<!-- ✓ Responsive images -->
<img
  srcset="image-small.jpg 480w, image-large.jpg 1024w"
  sizes="(max-width: 600px) 480px, 1024px"
  src="image-large.jpg"
  alt="Description"
/>

<!-- ✓ Picture element for format negotiation -->
<picture>
  <source srcset="image.webp" type="image/webp" />
  <source srcset="image.jpg" type="image/jpeg" />
  <img src="image.jpg" alt="Description" />
</picture>

<!-- ✓ Lazy load below-the-fold images -->
<img src="image.jpg" loading="lazy" />
```

### Store-Level Optimizations

```typescript
// ✓ Batch state updates
const updateUserProfile = async (changes: Partial<User>) => {
  try {
    const updated = await userService.update(changes);
    // Update all changes at once, not field-by-field
    Object.assign(user.value, updated);
  } catch (err) {
    // Handle error
  }
};

// ✓ Use getters for derived data (never store computed values)
const discountedPrice = computed(() =>
  product.value.price * (1 - product.value.discount)
);

// ✓ Implement cache invalidation strategy
const cacheTimestamp = ref<number | null>(null);
const cacheValidityMs = 5 * 60 * 1000; // 5 minutes

const shouldFetch = computed(() => {
  if (!cacheTimestamp.value) return true;
  return Date.now() - cacheTimestamp.value > cacheValidityMs;
});
```

### Bundle Size Monitoring

- Profile bundle with `npm run analyze`
- Set size budgets for chunks
- Use dynamic imports for large features
- Tree-shake unused code (proper ES module syntax)

---

## Architecture Decision Records

For significant architectural decisions, document them in **Architecture Decision Records (ADRs)**.

**Good candidates for ADRs:**
- Framework or tool choices (Vue vs React, Pinia vs Vuex)
- Architecture patterns (container/presentational, feature-based)
- State management strategy (centralized vs distributed)
- Styling approach (Tailwind vs CSS Modules)
- API integration patterns (REST vs GraphQL)
- Testing strategy (unit vs integration focus)
- Deployment & build strategy (code splitting, versioning)

**ADR Template:**
```markdown
# ADR-001: Use Composition API for Vue Components

## Status
Accepted

## Context
We needed to choose between Options API and Composition API for our Vue 3 codebase.

## Decision
We will use Composition API exclusively for all new components and gradually migrate
existing Options API code.

## Rationale
- Better code organization for large components
- Easier to compose and reuse logic
- Superior TypeScript support
- Industry standard for Vue 3

## Consequences
- Team members need training on Composition API patterns
- Refactoring of legacy Options API components
- Better long-term maintainability
```

---

## Appendix: Quick Reference

### Component Design Checklist

```
□ Single responsibility
□ Props fully typed
□ Independent operation
□ Events emitted for interactions
□ Local state for UI only
□ Handles loading/error/empty
□ WCAG 2.1 AA accessibility
□ < 500 lines
□ Testable design
□ Documented
□ Tailwind styling
```

### Data Flow Checklist

```
□ Data: parent → child (props)
□ Interactions: child → parent (emits)
□ No sibling-to-sibling direct communication
□ Complex flows use stores
□ Prop drilling avoided (max 2-3 levels)
□ No two-way binding (except forms)
□ Props never mutated in child
□ Event handlers in parent
```

### State Management Checklist

```
□ UI state in components (ref)
□ App state in Pinia stores
□ Derived data in computed properties
□ Async operations in store actions
□ State updates predictable & traceable
□ Store organized by feature/domain
□ Persistence intentional & documented
□ Error states managed
```

### Code Quality Checklist

```
□ TypeScript strict mode
□ No any types
□ Explicit return types
□ Prettier formatted
□ Props & emits typed
□ JSDoc on public API
□ Comments explain "why"
□ Naming conventions followed
□ No unused imports
□ Tests included
```

### File Organization Checklist

```
□ Feature-based structure
□ Naming conventions consistent
□ Clear shared vs feature separation
□ Services layer for API
□ Utilities organized
□ Types centralized
□ README documents structure
□ New devs understand layout
□ Directory depth reasonable
□ Related files colocated
```

---

## Additional Resources

- [Vue 3 Documentation](https://vuejs.org/guide/)
- [Pinia Documentation](https://pinia.vuejs.org/)
- [Tailwind CSS v4](https://tailwindcss.com/)
- [Vitest Documentation](https://vitest.dev/)
- [Nuxt 3 Documentation](https://nuxt.com/docs)
- [Zod Validation](https://zod.dev/)
- [day.js Documentation](https://day.js.org/)
- [WCAG 2.1 Guidelines](https://www.w3.org/WAI/WCAG21/quickref/)

---

**Document Version:** 1.0  
**Last Updated:** November 2025  
**Maintained by:** Engineering Team  
**Next Review:** November 2026

---
> Source: [junaidrasheed/vue3-senior-engineering-skill](https://github.com/junaidrasheed/vue3-senior-engineering-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
