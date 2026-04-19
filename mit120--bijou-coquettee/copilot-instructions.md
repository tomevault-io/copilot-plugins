## bijou-coquettee

> This is a **Medusa v2** e-commerce platform consisting of:

# Bijou Coquettee - Medusa.js E-commerce Platform
# Cursor AI Rules & Development Guidelines

## 🏗️ Project Structure Overview

This is a **Medusa v2** e-commerce platform consisting of:
1. **Backend**: `bijou-coquettee/` - Medusa.js application server
2. **Storefront**: `bijou-coquettee-storefront/` - Next.js 14+ App Router storefront

---

## 📚 Primary Documentation Reference

**Official Docs**: https://docs.medusajs.com/
**Version**: v2.11.1

### Key Documentation Sections:
- **Framework Guide**: https://docs.medusajs.com/learn
- **API Routes**: https://docs.medusajs.com/learn/basics/api-routes
- **Workflows**: https://docs.medusajs.com/learn/basics/workflows
- **Data Models**: https://docs.medusajs.com/learn/basics/data-models
- **Modules**: https://docs.medusajs.com/learn/basics/modules
- **Admin Customization**: https://docs.medusajs.com/learn/customization/customize-admin
- **Commerce Modules**: https://docs.medusajs.com/resources/references
- **Storefront Development**: https://docs.medusajs.com/resources/storefront-development

---

## 🎯 Backend Development Rules (bijou-coquettee/)

### 1. API Routes (`src/api/`)

**Structure**:
- `src/api/admin/` - Admin API endpoints (requires authentication)
- `src/api/store/` - Storefront API endpoints (public/customer)

**Route Creation Pattern**:
```typescript
// src/api/[admin|store]/[route-name]/route.ts
import { MedusaRequest, MedusaResponse } from "@medusajs/framework/http"

export async function GET(
  req: MedusaRequest,
  res: MedusaResponse
) {
  // Use dependency injection
  const query = req.scope.resolve("query")
  
  // Your logic here
  
  res.json({ data: result })
}
```

**Rules**:
- Always use `MedusaRequest` and `MedusaResponse` types
- Use dependency injection via `req.scope.resolve()`
- Follow REST conventions (GET, POST, PUT, DELETE, PATCH)
- Admin routes should validate admin authentication
- Store routes should handle guest and authenticated customers

**Documentation**: https://docs.medusajs.com/learn/basics/api-routes

---

### 2. Workflows (`src/workflows/`)

**Purpose**: Orchestrate complex business logic with rollback support

**Workflow Creation Pattern**:
```typescript
import { createWorkflow, WorkflowResponse } from "@medusajs/framework/workflows-sdk"

export const myCustomWorkflow = createWorkflow(
  "my-custom-workflow",
  function (input: InputType) {
    // Define steps
    const step1Result = step1(input)
    const step2Result = step2(step1Result)
    
    return new WorkflowResponse({
      result: step2Result
    })
  }
)
```

**Rules**:
- Workflows should be composable and reusable
- Each step should have compensation (rollback) logic
- Use built-in workflows when available
- Name workflows descriptively: `[entity]-[action]-workflow`
- Store workflows in `src/workflows/[feature]/`

**Documentation**: 
- https://docs.medusajs.com/learn/basics/workflows
- https://docs.medusajs.com/resources/references/workflows-sdk

---

### 3. Data Models (`src/modules/`)

**Data Model Definition**:
```typescript
import { model } from "@medusajs/framework/utils"

export const MyModel = model.define("my_model", {
  id: model.id().primaryKey(),
  name: model.text(),
  description: model.text().nullable(),
  // ... other fields
})
```

**Rules**:
- Use Data Model Language (DML) for defining models
- Follow snake_case for database column names
- Use camelCase for TypeScript property names
- Always define proper relationships (hasOne, hasMany, belongsTo)
- Add indexes for frequently queried fields
- Use enums for status fields

**Documentation**: 
- https://docs.medusajs.com/learn/basics/data-models
- https://docs.medusajs.com/resources/references/data-model

---

### 4. Modules (`src/modules/`)

**Module Structure**:
```
src/modules/[module-name]/
├── models/
│   └── [model-name].ts
├── services/
│   └── [service-name].ts
├── workflows/
│   └── [workflow-name].ts
├── migrations/
│   └── [timestamp]-[description].ts
└── index.ts
```

**Service Pattern**:
```typescript
import { MedusaService } from "@medusajs/framework/utils"

class MyModuleService extends MedusaService({
  MyModel,
}) {
  async customMethod(id: string) {
    // Your custom logic
  }
}

export default MyModuleService
```

**Rules**:
- One module per business domain
- Export module definition in `index.ts`
- Services should extend `MedusaService` for CRUD operations
- Use transactions for multi-step operations
- Module names should be lowercase-hyphenated

**Documentation**: https://docs.medusajs.com/learn/basics/modules

---

### 5. Links (`src/links/`)

**Purpose**: Define relationships between modules

**Link Definition**:
```typescript
import { defineLink } from "@medusajs/framework/utils"
import MyModule from "../modules/my-module"
import ProductModule from "@medusajs/framework/product"

export default defineLink(
  MyModule.linkable.myModel,
  ProductModule.linkable.product
)
```

**Rules**:
- Links enable cross-module queries
- Use for relationships between different modules
- Define in `src/links/` directory
- Always specify both sides of the relationship

**Documentation**: https://docs.medusajs.com/learn/basics/modules/module-links

---

### 6. Subscribers (`src/subscribers/`)

**Purpose**: React to domain events

**Subscriber Pattern**:
```typescript
import type { SubscriberConfig } from "@medusajs/framework"

export default async function orderPlacedHandler({ event, container }) {
  // React to event
}

export const config: SubscriberConfig = {
  event: "order.placed",
}
```

**Rules**:
- Subscribers should be idempotent
- Use for side effects (notifications, integrations)
- Avoid heavy processing (use jobs instead)
- Always handle errors gracefully

**Documentation**: https://docs.medusajs.com/learn/basics/events-and-subscribers

---

### 7. Jobs (`src/jobs/`)

**Purpose**: Scheduled or background tasks

**Job Pattern**:
```typescript
import type { MedusaContainer } from "@medusajs/framework/types"

export default async function myScheduledJob(
  container: MedusaContainer
) {
  // Your job logic
}

export const config = {
  name: "my-scheduled-job",
  schedule: "0 0 * * *", // Cron expression
}
```

**Rules**:
- Use for long-running tasks
- Use for scheduled operations (cleanup, reports)
- Always log job execution
- Handle failures with retries

**Documentation**: https://docs.medusajs.com/learn/basics/scheduled-jobs

---

### 8. Admin Customization (`src/admin/`)

**Structure**:
- `src/admin/widgets/` - Custom admin widgets
- `src/admin/routes/` - Custom admin pages
- `src/admin/i18n/` - Internationalization

**Widget Pattern**:
```typescript
import { defineWidgetConfig } from "@medusajs/admin-sdk"

const MyWidget = () => {
  return <div>My Custom Widget</div>
}

export const config = defineWidgetConfig({
  zone: "product.details.after",
})

export default MyWidget
```

**Rules**:
- Use provided UI components from `@medusajs/ui`
- Follow React best practices
- Keep widgets focused and single-purpose
- Use injection zones for placement

**Documentation**: https://docs.medusajs.com/learn/customization/customize-admin

---

### 9. Configuration (`medusa-config.ts`)

**Key Configuration Areas**:
- Database connection
- Redis configuration
- Admin CORS settings
- Store CORS settings
- Plugins and modules
- Feature flags

**Rules**:
- Use environment variables for sensitive data
- Document all custom configuration
- Follow security best practices for CORS
- Enable required modules in `modules` object

---

## 🎨 Storefront Development Rules (bijou-coquettee-storefront/)

### 1. Project Architecture

**Framework**: Next.js 14+ with App Router
**Styling**: Tailwind CSS
**State Management**: React Context + Server Components

**Directory Structure**:
```
src/
├── app/                    # Next.js App Router pages
│   ├── [countryCode]/     # Country-specific routes
│   │   ├── (checkout)/    # Checkout flow (route group)
│   │   └── (main)/        # Main store pages (route group)
│   └── layout.tsx         # Root layout
├── modules/               # Feature modules
├── lib/                   # Utilities and data fetching
├── styles/                # Global styles
└── types/                 # TypeScript types
```

---

### 2. Module Pattern (`src/modules/`)

**Module Structure**:
```
modules/[feature]/
├── components/           # Feature-specific components
│   └── [component]/
│       └── index.tsx
└── templates/           # Page templates
    └── index.tsx
```

**Rules**:
- Each feature should be self-contained in its module
- Templates compose components into full pages
- Components should be focused on single responsibility
- Use barrel exports (index.tsx) for cleaner imports

**Existing Modules**:
- `account/` - Customer account management
- `cart/` - Shopping cart functionality
- `checkout/` - Checkout flow
- `products/` - Product display and details
- `collections/` - Product collections
- `categories/` - Product categories
- `order/` - Order confirmation and details
- `layout/` - Global layout components (nav, footer)
- `common/` - Shared components and icons

---

### 3. Data Fetching (`src/lib/data/`)

**Server-Side Data Fetching Pattern**:
```typescript
import { cookies } from "next/headers"
import { getAuthHeaders } from "./cookies"

export async function getProduct(id: string) {
  const headers = {
    ...getAuthHeaders(),
    "x-publishable-api-key": process.env.NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY!,
  }
  
  const response = await fetch(
    `${process.env.NEXT_PUBLIC_MEDUSA_BACKEND_URL}/store/products/${id}`,
    { 
      headers,
      next: { tags: [`product:${id}`] } // For revalidation
    }
  )
  
  return response.json()
}
```

**Rules**:
- Use Server Components for data fetching when possible
- Include auth headers for authenticated requests
- Use Next.js cache tags for revalidation
- Handle errors gracefully with try-catch
- Use TypeScript types for API responses

**Data Modules**:
- `cart.ts` - Cart operations
- `products.ts` - Product queries
- `collections.ts` - Collection queries
- `categories.ts` - Category queries
- `customer.ts` - Customer data
- `orders.ts` - Order operations
- `regions.ts` - Region and country data
- `payment.ts` - Payment processing
- `fulfillment.ts` - Shipping and fulfillment

---

### 4. Route Organization (`src/app/`)

**Route Groups**:
- `(checkout)/` - Isolated checkout experience
- `(main)/` - Main store with shared navigation

**Country Code Routing**:
- All routes are prefixed with `[countryCode]` for multi-region support
- Middleware handles country detection and routing

**Rules**:
- Use route groups for shared layouts
- Keep page components focused on data fetching and composition
- Use loading.tsx for loading states
- Use error.tsx for error boundaries
- Implement not-found.tsx for 404 pages

---

### 5. Styling Guidelines

**Tailwind CSS**:
```typescript
// Good: Semantic utility classes
<div className="flex items-center justify-between p-4 bg-white rounded-lg shadow-md">

// Avoid: Inline styles
<div style={{ display: 'flex', padding: '1rem' }}>
```

**Rules**:
- Use Tailwind utilities for all styling
- Keep responsive design in mind (mobile-first)
- Use `globals.css` for base styles and CSS variables
- Follow consistent spacing (p-4, gap-4, etc.)
- Use semantic color names from Tailwind config

---

### 6. Component Best Practices

**Server Components (Default)**:
```typescript
// app/products/[id]/page.tsx
export default async function ProductPage({ params }: { params: { id: string } }) {
  const product = await getProduct(params.id)
  return <ProductTemplate product={product} />
}
```

**Client Components** (when needed):
```typescript
"use client"

import { useState } from "react"

export default function InteractiveComponent() {
  const [state, setState] = useState()
  // Component logic
}
```

**Rules**:
- Prefer Server Components by default
- Use Client Components only when needed (interactivity, hooks, browser APIs)
- Keep Client Components small and focused
- Pass data from Server to Client Components via props
- Use TypeScript for all components

---

### 7. State Management

**Context Usage** (`src/lib/context/`):
- `modal-context.tsx` - Modal state management

**Rules**:
- Use Context sparingly (mostly for global UI state)
- Prefer Server Components + URL state for most data
- Use URL search params for filterable lists
- Cookies for cart/session management

---

### 8. Middleware (`src/middleware.ts`)

**Purpose**:
- Country/region detection
- Redirect logic
- Cart ID management

**Rules**:
- Keep middleware lightweight
- Use for route protection and redirection
- Handle region-based routing

---

### 9. Environment Variables

**Required Variables**:
```bash
NEXT_PUBLIC_MEDUSA_BACKEND_URL=http://localhost:9000
NEXT_PUBLIC_MEDUSA_PUBLISHABLE_KEY=pk_...
NEXT_PUBLIC_DEFAULT_REGION=us
NEXT_PUBLIC_STRIPE_KEY=pk_test_... # If using Stripe
```

**Rules**:
- Use `NEXT_PUBLIC_` prefix for client-side variables
- Keep sensitive keys server-side only
- Document all required variables in env.example
- Use `src/lib/util/env.ts` for env validation

---

## 🔧 Development Workflow

### 1. Starting Development

**Backend**:
```bash
cd bijou-coquettee
npm install
npm run dev
# Runs on http://localhost:9000
```

**Storefront**:
```bash
cd bijou-coquettee-storefront
npm install
npm run dev
# Runs on http://localhost:3000
```

### 2. Database Migrations

```bash
cd bijou-coquettee
npx medusa db:migrate
npx medusa db:seed # Seed initial data
```

### 3. Building for Production

**Backend**:
```bash
npm run build
npm run start
```

**Storefront**:
```bash
npm run build
npm run start
```

---

## 🧪 Testing

### Integration Tests (`bijou-coquettee/integration-tests/`)

**Pattern**:
```typescript
import { medusaIntegrationTestRunner } from "@medusajs/test-utils"

medusaIntegrationTestRunner({
  testSuite: ({ api, getContainer }) => {
    describe("Feature Tests", () => {
      it("should do something", async () => {
        const response = await api.get("/store/endpoint")
        expect(response.status).toBe(200)
      })
    })
  }
})
```

**Rules**:
- Write integration tests for custom API routes
- Test workflows end-to-end
- Mock external services
- Use test database

---

## 📝 Code Style & Conventions

### TypeScript
- Use strict mode
- Define interfaces for all API responses
- Use type inference when obvious
- Avoid `any` type

### Naming Conventions
- Files: kebab-case (`my-component.tsx`)
- Components: PascalCase (`MyComponent`)
- Functions: camelCase (`myFunction`)
- Constants: UPPER_SNAKE_CASE (`MY_CONSTANT`)
- Database tables: snake_case (`my_table`)

### Imports
- Group imports: external → internal → relative
- Use absolute imports when possible
- Avoid circular dependencies

---

## 🚨 Common Pitfalls & Solutions

### 1. "Module not found" Errors
- Check `tsconfig.json` paths
- Ensure modules are registered in `medusa-config.ts`
- Run `npm install` in both directories

### 2. CORS Issues
- Configure CORS in `medusa-config.ts`
- Add storefront origin to admin and store CORS

### 3. Database Connection Issues
- Verify PostgreSQL is running
- Check database credentials in `.env`
- Run migrations: `npx medusa db:migrate`

### 4. Cache Issues in Storefront
- Clear `.next` folder
- Check cache tags in data fetching functions
- Use `revalidatePath` or `revalidateTag` after mutations

---

## 📖 Quick Reference: When to Search Documentation

**Search Medusa Docs when you need to**:
1. ✅ Create a new API route → Search: "medusa api routes"
2. ✅ Build a workflow → Search: "medusa workflows"
3. ✅ Create a data model → Search: "medusa data models"
4. ✅ Build a custom module → Search: "medusa modules"
5. ✅ Link modules together → Search: "medusa module links"
6. ✅ Subscribe to events → Search: "medusa events subscribers"
7. ✅ Customize admin dashboard → Search: "medusa customize admin"
8. ✅ Integrate third-party services → Search: "medusa integrations"
9. ✅ Use commerce modules (Product, Cart, etc.) → Search: "[module name] module medusa"
10. ✅ Handle payments → Search: "medusa payment module"
11. ✅ Manage inventory → Search: "medusa inventory module"
12. ✅ Set up multi-region → Search: "medusa region module"
13. ✅ Implement promotions → Search: "medusa promotion module"
14. ✅ Build marketplace features → Search: "medusa marketplace recipe"
15. ✅ Add digital products → Search: "medusa digital products recipe"

**Documentation Structure**:
- **Get Started**: Basic setup and introduction
- **Learn**: Core concepts and framework basics
- **Build**: Recipes and tutorials for specific use cases
- **Reference**: API reference, data models, workflows
- **Resources**: Additional tools and integrations

---

## 🎯 AI Assistant Instructions

When helping with this project:

1. **Always reference Medusa v2 documentation** - The project uses Medusa v2, not v1
2. **Check existing patterns first** - Look at existing files before creating new ones
3. **Follow module boundaries** - Keep admin, store, and custom logic separated
4. **Use TypeScript strictly** - No implicit any types
5. **Consider the full stack** - Changes often affect both backend and storefront
6. **Test integration points** - API routes should be tested with storefront consumption in mind
7. **Document custom features** - Add JSDoc comments for custom APIs and workflows
8. **Security first** - Always validate input, check authentication, handle errors

### Suggesting Changes:
- Provide complete file contents, not partial snippets
- Explain the Medusa concepts being used
- Link to relevant documentation
- Consider backward compatibility
- Suggest testing approach

### When Uncertain:
- State what you know vs. what you're inferring
- Reference specific documentation pages to verify
- Suggest searching the Medusa docs for clarification
- Ask clarifying questions about business requirements

---

## 🔗 Essential Links

- **Main Documentation**: https://docs.medusajs.com/
- **GitHub Repository**: https://github.com/medusajs/medusa
- **Discord Community**: https://discord.gg/medusajs
- **Admin UI Components**: https://docs.medusajs.com/ui
- **Next.js Starter**: https://docs.medusajs.com/resources/nextjs-starter
- **Recipes**: https://docs.medusajs.com/resources/recipes
- **API Reference**: https://docs.medusajs.com/api
- **Changelog**: https://docs.medusajs.com/changelog

---

## 📦 Key Dependencies

**Backend**:
- `@medusajs/framework` - Core framework
- `@medusajs/medusa` - Medusa server
- PostgreSQL - Primary database
- Redis - Caching and job queue

**Storefront**:
- Next.js 14+ - React framework
- Tailwind CSS - Styling
- TypeScript - Type safety

---

**Last Updated**: 2025-10-27
**Medusa Version**: v2.11.1
**Next.js Version**: 14+

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MIT120) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
