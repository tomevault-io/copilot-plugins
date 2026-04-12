## crewbolt

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Package Manager**: This project uses Bun (not npm/yarn)
- **Dev server**: `npm run dev` (uses Turbopack for faster builds)
- **Build**: `npm run build` (production build with Turbopack)
- **Production**: `npm run start`
- **Lint**: `npm run lint` (ESLint with Next.js config)

## Production Application Guidelines

**BUSINESS-FOCUSED DEVELOPMENT**: This is a real production application, not a demo or template project.

### What to Build
- **User management systems** for real business needs
- **Analytics dashboards** with actionable business insights
- **Authentication flows** for actual user onboarding
- **Data management interfaces** for real business operations
- **Notification systems** for user engagement
- **Settings and preferences** for user customization
- **API integrations** for business tools and services

### What NOT to Build
- Demo pages or example routes (`/demo`, `/test`, `/example`)
- Placeholder content or fake data displays
- "Kitchen sink" component showcases
- Tutorial or documentation routes within the app
- Sample workflows that don't serve real users
- Proof-of-concept features without business value

## Design Philosophy

**PRODUCTION APPLICATION FOCUS**: This is a production application - only build real, production-ready features that users will actually use.

**SHADCN ULTRATHINK**: This project follows shadcn/ui design principles exclusively. All UI decisions should align with shadcn aesthetic and patterns:

- **Design System**: Use only shadcn/ui components, variants, and patterns
- **Color Palette**: Stick to shadcn color tokens and semantic naming (background, foreground, muted, accent, etc.)
- **Typography**: Follow shadcn typography scale and font choices
- **Spacing & Layout**: Use shadcn spacing patterns and layout components
- **Component Composition**: Build complex UIs by composing shadcn primitives
- **Accessibility**: Leverage Radix UI's built-in accessibility features
- **MINIMALISTIC DESIGN**: No borders (`border-0`), favor subtle backgrounds (`bg-muted/20`, `bg-muted/30`)
- **COMPACT SIZING**: Ultra-compact desktop (h-8 buttons, p-3/p-4 cards, w-64 sidebar), mobile touch targets (44px)
- **Visual Hierarchy**: Use background colors, spacing, and typography instead of borders
- **Dense Layout**: Reduced padding and spacing for professional, space-efficient interface

## ULTRATHINK ORGANIZATION

**Extreme Focus on Organization**: This project follows top-tier open source project structures (Vercel, shadcn, etc.) with obsessive attention to scalability and maintainability.

### Folder Structure Standards

```
/
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Route group: Authentication
│   │   ├── layout.tsx            # Auth-specific layout
│   │   ├── login/page.tsx
│   │   └── register/page.tsx
│   ├── (dashboard)/              # Route group: Protected areas
│   │   ├── layout.tsx
│   │   ├── analytics/
│   │   │   ├── components/       # Feature-specific components
│   │   │   ├── hooks/            # Feature-specific hooks
│   │   │   ├── types/            # Feature-specific types
│   │   │   ├── page.tsx
│   │   │   ├── loading.tsx
│   │   │   └── error.tsx
│   │   └── users/
│   ├── (marketing)/              # Route group: Public pages
│   │   ├── layout.tsx
│   │   ├── page.tsx              # Homepage
│   │   ├── about/page.tsx
│   │   └── pricing/page.tsx
│   ├── api/                      # API routes
│   │   ├── auth/login/route.ts
│   │   ├── users/route.ts
│   │   └── users/[id]/route.ts
│   ├── globals.css
│   ├── layout.tsx                # Root layout
│   └── page.tsx                  # Root page
├── components/                   # Global components
│   ├── ui/                       # shadcn/ui components (atomic)
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   └── index.ts              # Barrel exports
│   ├── layout/                   # Layout components (molecules)
│   │   ├── header.tsx
│   │   ├── navigation.tsx
│   │   └── footer.tsx
│   └── features/                 # Feature components (organisms)
│       ├── auth/
│       ├── dashboard/
│       └── commerce/
├── lib/                          # Utilities & configurations
│   ├── auth.ts
│   ├── db.ts
│   ├── validations.ts
│   └── utils.ts
├── hooks/                        # Custom React hooks
├── types/                        # TypeScript definitions
│   ├── global.ts
│   ├── auth.ts
│   └── api.ts
├── config/                       # Configuration files
│   ├── site.ts
│   └── database.ts
└── public/                       # Static assets
    ├── images/
    ├── icons/
    └── favicon.ico
```

### Naming Conventions (STRICT)

**Files & Components:**
- **Pages**: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `not-found.tsx`
- **API Routes**: `route.ts`
- **shadcn/ui Components**: `button.tsx`, `card.tsx`, `input.tsx` (kebab-case)
- **Custom Components**: `UserProfile.tsx`, `ProductCard.tsx` (PascalCase)
- **Feature Components**: `AuthForm.tsx`, `DashboardNav.tsx` (PascalCase)

**Directories:**
- **Route Groups**: `(auth)`, `(dashboard)`, `(marketing)` (parentheses)
- **Routes**: `user-profile`, `product-details` (kebab-case)
- **Features**: `userManagement`, `productCatalog` (camelCase)
- **Private Folders**: `_components`, `_utils`, `_lib` (underscore prefix)

**Utilities & Config:**
- **Utilities**: `formatDate.ts`, `validateEmail.ts` (camelCase)
- **Config**: `next.config.js`, `tailwind.config.js` (kebab-case)

### Organization Patterns

**1. Feature-Based Organization:**
```
app/(dashboard)/analytics/
├── components/               # Feature-specific components
│   ├── MetricsCard.tsx
│   ├── AnalyticsChart.tsx
│   └── index.ts
├── hooks/                   # Feature-specific hooks
│   └── useAnalytics.ts
├── types/                   # Feature-specific types
│   └── analytics.ts
├── lib/                     # Feature-specific utilities
│   └── analytics-utils.ts
├── page.tsx
├── loading.tsx
└── error.tsx
```

**2. Atomic Design Component Hierarchy:**
```
components/
├── ui/                      # Atoms (basic elements)
│   ├── button.tsx
│   ├── input.tsx
│   └── card.tsx
├── layout/                  # Molecules (combinations)
│   ├── header.tsx
│   └── navigation.tsx
└── features/                # Organisms (complex)
    ├── auth/LoginForm.tsx
    └── commerce/ProductGrid.tsx
```

**3. Barrel Exports for Clean Imports:**
```typescript
// components/ui/index.ts
export { Button } from './button'
export { Card } from './card'
export { Input } from './input'

// Usage
import { Button, Card, Input } from '@/components/ui'
```

**4. Testing Organization:**
```
components/
├── ui/
│   ├── button.tsx
│   ├── button.test.tsx          # Unit tests
│   └── button.stories.tsx       # Storybook stories
├── features/
│   └── auth/
│       ├── LoginForm.tsx
│       ├── LoginForm.test.tsx
│       └── __tests__/           # Complex test suites
│           └── integration.test.tsx
└── __tests__/                   # Global tests
    ├── setup.ts
    └── utils.test.tsx
```

**5. Asset Co-location:**
```
public/
├── images/
│   ├── logos/                   # Brand assets
│   ├── heroes/                  # Hero images
│   └── avatars/                 # User avatars
├── icons/
│   ├── brand/                   # Brand icons
│   └── social/                  # Social media icons
└── favicon.ico

components/ProductCard/
├── ProductCard.tsx
├── ProductCard.test.tsx
├── ProductCard.stories.tsx
├── assets/                      # Component-specific assets
│   └── placeholder.svg
└── index.ts                     # Barrel export
```

### ULTRATHINK Rules (MANDATORY)

**1. Consistency Over Convention:**
- Choose patterns and stick to them project-wide
- Never mix organizational approaches within the same feature

**2. Feature-First Thinking:**
- Organize by business features, not technical layers
- Keep related functionality co-located

**3. Progressive Disclosure:**
- Start simple, add complexity as needed
- Use private folders (`_folder`) for internal organization

**4. Clear Boundaries:**
- Use route groups `(name)` for logical organization
- Separate public/private routes clearly

**5. Scalable Imports:**
- Always use barrel exports for component groups
- Leverage path aliases for clean import statements

**6. Type Safety First:**
- Organize types by feature and domain
- Co-locate types with their related components

## Project Architecture

**Tech Stack**: Next.js 15.5.4 with App Router, React 19, TypeScript, Tailwind CSS v4, Bun runtime

### Path Aliases Configuration
```typescript
// tsconfig.json paths
"@/*": ["./src/*"]
"@/ui": ["./components/ui"]
"@/lib/*": ["./lib/*"]
"@/types/*": ["./types/*"]
"@/hooks/*": ["./hooks/*"]
"@/config/*": ["./config/*"]
```

### Component System
- Uses shadcn/ui "New York" style with Radix UI primitives
- All components support dark/light theming via CSS custom properties
- Component variants managed with `class-variance-authority`
- Icons from `lucide-react`

### Key Configuration Files
- **TypeScript**: `tsconfig.json` with strict mode, path aliases
- **shadcn/ui**: `components.json` defines component locations and styling
- **Tailwind**: v4 configuration embedded in `globals.css`
- **ESLint**: Modern flat config in `eslint.config.mjs`

## Development Patterns

### Production-First Development
- **REAL FEATURES ONLY**: Build actual business functionality, not demos or examples
- **USER-FOCUSED**: Every feature must solve a real user problem or business need
- **PRODUCTION READY**: All code must be production-quality from day one
- **NO PLACEHOLDER CONTENT**: Use real data, real workflows, real use cases
- **BUSINESS VALUE**: Each feature must provide clear business or user value

### Component Reuse Strategy (ULTRATHINK MANDATORY)

**EXTREME ANTI-DUPLICATION RULES**:
- **MAXIMUM ONE COMPONENT PER TYPE**: Never create multiple headers, sidebars, navbars, or similar components
- **SINGLE SOURCE OF TRUTH**: One layout component per feature area (one header, one sidebar, one footer, etc.)
- **SEARCH BEFORE CREATE**: Always use `Glob` and `Grep` tools to find existing components before creating new ones
- **AUDIT FIRST**: Before any new component, audit existing codebase for similar functionality
- **DELETE UNUSED**: Aggressively remove any component variants that aren't actively used in production routes

**MANDATORY CHECKS BEFORE COMPONENT CREATION**:
1. Run `Glob "components/**/*{header,sidebar,nav,layout}*"` to find existing components
2. Check ALL layout files (`Glob "**/layout.tsx"`) to see what's currently imported
3. If similar functionality exists, EXTEND the existing component instead of creating new
4. If multiple variants exist, CONSOLIDATE into single component with props/variants
5. NEVER create component names ending in `-v2`, `-new`, `-alt`, `-shopify`, `-ios`, etc.

**COMPONENT HIERARCHY ENFORCEMENT**:
- **Headers**: `DashboardHeader` ONLY - extends via props, never duplicate
- **Sidebars**: `SidebarDrill` ONLY - extends via props, never duplicate
- **Layouts**: Use route groups `(dashboard)` for organization, ONE layout per group
- **Forms**: Extend shadcn `Form` components, never create custom form wrappers
- **Cards**: Use shadcn `Card` only, extend via `CardHeader`, `CardContent`, `CardFooter`

**PRODUCTION ULTRATHINK RULES**:
- **ALWAYS use existing components** before creating new ones
- **Leverage component composition** to build complex UIs from simple parts
- **Extend existing components** through props and variants, not custom CSS
- **Check existing codebase** for similar functionality before implementing
- **Compose complex layouts** using `Card`, `Sheet`, `Accordion`, etc.
- **Follow shadcn examples** from docs for implementation patterns
- **Use shadcn color system**: `bg-background`, `text-foreground`, `bg-muted/20`, `bg-muted/30`
- **NO DEMO/TEST ROUTES**: Never create `/demo`, `/test`, `/example` routes for testing features
- **CONTINUOUS CLEANUP**: Regular audits to remove unused components and consolidate duplicates

### Advanced Sidebar System
- **ChatGPT-inspired design** with ultrathink minimalistic approach
- **Supports 100+ menu items** with virtual scrolling for performance
- **Toggle functionality** with smooth animations and mobile support
- **Instant search** with ⌘K keyboard shortcut and real-time filtering
- **Context menus** on hover with pin, star, edit, archive, delete actions
- **Smart categorization** with collapsible sections and item counts
- **Performance optimized** with virtual rendering for large lists
- **Mobile-first** with backdrop overlay and touch-optimized interactions
- **Keyboard navigation** with arrow keys, Enter/Space, and Escape shortcuts
- **Resizable width** with drag handle and min/max constraints

### Form Handling
- Use `react-hook-form` with `zod` for validation
- Build forms with shadcn components: `Form`, `FormField`, `FormItem`, `FormLabel`, `FormControl`, `FormMessage`
- Use `Input`, `Textarea`, `Select`, `Checkbox`, `RadioGroup` from shadcn/ui
- Follow shadcn form patterns and validation approaches

### Layout & Navigation
- **Cards**: Use `Card` component for content containers
- **Navigation**: Build with `NavigationMenu`, `Breadcrumb`, `Pagination`
- **Data Display**: Use `Table`, `DataTable`, `Badge`, `Avatar` components
- **Overlays**: Use `Sheet`, `Popover`, `HoverCard` for contextual content
- **Content Organization**: Use `Accordion`, `Collapsible` for expandable content

### Theming
- Dark/light mode via `next-themes` provider
- Use only shadcn semantic color tokens (never hex values or custom colors)
- All components automatically support theme switching
- Follow shadcn's color contrast and accessibility guidelines

## PLAYWRIGHT TESTING & MONITORING

### Playwright MCP Server Integration
- **Headless Mode Always**: Run Playwright in headless mode for continuous monitoring
- **Real-time Log Monitoring**: Monitor console logs, network requests, and performance metrics
- **Design Testing**: Automatically test visual consistency and layout responsiveness
- **Cross-browser Validation**: Ensure compatibility across different browsers and devices
- **Performance Tracking**: Monitor Core Web Vitals and loading times
- **NO TEST ROUTES**: Never create `/demo`, `/test`, or `/example` routes - use Playwright exclusively

### Testing Commands
```bash
# Run Playwright tests in headless mode (default)
npx playwright test

# Monitor specific features during development
npx playwright test --grep="sidebar|navigation|mobile"

# Performance validation
npx playwright test --grep="performance"

# Visual regression testing
npx playwright test --update-snapshots

# Mobile-specific testing
npx playwright test --project="Mobile Chrome"
```

### Production Testing Philosophy
- **Production Routes Only**: Test real application routes that users will actually use
- **Business Workflow Testing**: Validate real user journeys and business processes
- **Feature Validation**: Use Playwright to test actual production functionality
- **Design Consistency**: Automated visual regression testing on production features
- **Performance Monitoring**: Continuous Web Vitals validation on live features
- **Real User Simulation**: Test authentic user workflows that drive business value
- **Production Data Testing**: Use real data structures and business logic in tests

## MOBILE-FIRST APP EXPERIENCE

### Mobile-First Design Principles
- **Mobile-first development**: Design for mobile, enhance for desktop
- **Touch-optimized interactions**: Minimum 44px touch targets
- **Gesture support**: Swipe, pinch, and scroll interactions
- **Viewport meta tag**: Proper mobile viewport configuration
- **Responsive breakpoints**: `sm:`, `md:`, `lg:`, `xl:`, `2xl:` in order of importance

### App-Like Experience Requirements
- **INSTANT LOADING**: Pages must appear immediately - no large loading screens
- **Component-level loading**: Use loading states within buttons, cards, and individual components
- **Stateful buttons**: Buttons show loading state during actions (`loading` prop)
- **Skeleton loading**: Use `Skeleton` component for individual content areas
- **Smooth animations**: 60fps animations using CSS transforms (avoid heavy JavaScript)
- **Native-like navigation**: Bottom navigation, floating action buttons
- **Progressive Web App**: Service worker, manifest.json, offline capabilities
- **Touch feedback**: Visual feedback for all interactive elements
- **Safe areas**: Handle device notches and safe areas properly

### Responsive Design Standards
```typescript
// Use mobile-first responsive classes
<div className="w-full md:w-1/2 lg:w-1/3">
  <Card className="p-4 md:p-6 lg:p-8">
    <Button className="w-full md:w-auto">Action</Button>
  </Card>
</div>

// Always test these breakpoints:
// Mobile: 375px - 767px
// Tablet: 768px - 1023px
// Desktop: 1024px+
```

## SPEED OVER EVERYTHING

### Performance Optimization (NO CACHING)
- **SPEED FIRST**: Prioritize speed over all other concerns (caching comes at project end)
- **Bundle size**: Keep JavaScript bundles < 150KB gzipped (stricter than before)
- **Zero large loading pages**: Never show full-page loading screens
- **Instant route transitions**: Pages must render immediately with skeleton content
- **Component-level loading**: Individual components handle their own loading states
- **Image optimization**: Use Next.js Image with priority loading for above-fold content
- **Aggressive code splitting**: Split at component and route level for minimal initial bundles
- **Lazy loading**: Implement for all non-critical components and images
- **Preload critical resources**: Preload fonts, critical CSS, and above-fold images

### Loading State Strategy
```typescript
// Stateful Button Example
<Button loading={isSubmitting} disabled={isSubmitting}>
  {isSubmitting ? "Saving..." : "Save Changes"}
</Button>

// Component-level loading
<Card>
  {isLoading ? (
    <Skeleton className="h-20 w-full" />
  ) : (
    <CardContent>{data}</CardContent>
  )}
</Card>

// Never do this - no full page loading
{/* ❌ AVOID */}
{isLoading ? <FullPageSpinner /> : <PageContent />}

{/* ✅ DO THIS */}
<Layout>
  <Header />
  {isLoading ? <Skeleton className="h-96" /> : <MainContent />}
  <Footer />
</Layout>
```

## VERCEL-LEVEL SPEED TECHNIQUES

### 1. Edge Candidate Generation & Streaming
```typescript
// Generate candidates as user types, stream results immediately
const useCandidateStream = (query: string) => {
  const [results, setResults] = useState<Result[]>([])

  useEffect(() => {
    if (!query) return

    // Generate candidates immediately
    const exactCandidates = generateExactMatches(query)
    const variantCandidates = generateVariants(query)

    // Stream results as they resolve
    Promise.allSettled([
      checkCandidates(exactCandidates),
      checkCandidates(variantCandidates)
    ]).then(streamResults => {
      streamResults.forEach(result => {
        if (result.status === 'fulfilled') {
          setResults(prev => [...prev, ...result.value])
        }
      })
    })
  }, [query])

  return results
}
```

### 2. Optimistic UI with Instant Feedback
```typescript
// Show optimistic results immediately, reconcile later
const SearchResults = () => {
  const [optimisticResults, setOptimisticResults] = useState([])
  const [confirmedResults, setConfirmedResults] = useState([])

  const handleSearch = (query: string) => {
    // Instant optimistic results
    setOptimisticResults(generateOptimisticResults(query))

    // Background verification
    verifyResults(query).then(confirmed => {
      setConfirmedResults(confirmed)
      // Reconcile any differences
      reconcileOptimisticState(optimisticResults, confirmed)
    })
  }

  return (
    <div>
      {optimisticResults.map(result => (
        <Card key={result.id} className={result.verified ? '' : 'opacity-80'}>
          <Badge variant={result.status === 'checking' ? 'secondary' : 'default'}>
            {result.status === 'checking' ? 'Checking...' : result.status}
          </Badge>
          {result.name}
        </Card>
      ))}
    </div>
  )
}
```

### 3. Structured Concurrency with Timeouts
```typescript
// Parallel processing with strict timeouts and cancellation
const useStructuredConcurrency = () => {
  const processWithTimeout = async <T>(
    operations: (() => Promise<T>)[],
    timeoutMs = 2000
  ) => {
    const controller = new AbortController()

    // Auto-timeout
    const timeout = setTimeout(() => controller.abort(), timeoutMs)

    try {
      // Run operations in parallel with cancellation support
      const results = await Promise.allSettled(
        operations.map(op =>
          op().catch(err => {
            if (controller.signal.aborted) throw new Error('Timeout')
            throw err
          })
        )
      )

      clearTimeout(timeout)
      return results.filter(r => r.status === 'fulfilled')
    } catch (error) {
      controller.abort()
      throw error
    }
  }

  return { processWithTimeout }
}
```

### 4. Partitioned Batching by Service
```typescript
// Group requests by endpoint/service for optimal batching
const batchRequestsByService = (requests: Request[]) => {
  const partitions = new Map<string, Request[]>()

  requests.forEach(req => {
    const serviceKey = getServiceKey(req)
    if (!partitions.has(serviceKey)) {
      partitions.set(serviceKey, [])
    }
    partitions.get(serviceKey)!.push(req)
  })

  // Process each partition concurrently with service-specific limits
  return Promise.allSettled(
    Array.from(partitions.entries()).map(([service, reqs]) =>
      processServiceBatch(service, reqs, getServiceLimits(service))
    )
  )
}
```

### 5. In-Memory Fast Gates (Bloom Filter Pattern)
```typescript
// Fast "probably exists" checks before expensive operations
class FastGate {
  private knownExists = new Set<string>()
  private recentChecks = new Map<string, boolean>()
  private maxCacheSize = 10000

  async probablyExists(key: string): Promise<'likely' | 'unlikely' | 'unknown'> {
    // Instant positive check
    if (this.knownExists.has(key)) return 'likely'

    // Recent cache check
    if (this.recentChecks.has(key)) {
      return this.recentChecks.get(key) ? 'likely' : 'unlikely'
    }

    return 'unknown'
  }

  updateGate(key: string, exists: boolean) {
    if (exists) this.knownExists.add(key)

    this.recentChecks.set(key, exists)

    // LRU cleanup
    if (this.recentChecks.size > this.maxCacheSize) {
      const firstKey = this.recentChecks.keys().next().value
      this.recentChecks.delete(firstKey)
    }
  }
}
```

### 6. Circuit Breaker Pattern for Resilience
```typescript
// Prevent cascade failures from slow services
class CircuitBreaker {
  private failures = 0
  private lastFailureTime = 0
  private state: 'closed' | 'open' | 'half-open' = 'closed'

  async execute<T>(operation: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      if (Date.now() - this.lastFailureTime > 30000) { // 30s timeout
        this.state = 'half-open'
      } else {
        throw new Error('Circuit breaker OPEN')
      }
    }

    try {
      const result = await operation()
      this.onSuccess()
      return result
    } catch (error) {
      this.onFailure()
      throw error
    }
  }

  private onSuccess() {
    this.failures = 0
    this.state = 'closed'
  }

  private onFailure() {
    this.failures++
    this.lastFailureTime = Date.now()

    if (this.failures >= 5) {
      this.state = 'open'
    }
  }
}
```

### 7. Streaming Updates Pattern
```typescript
// Stream results as they arrive instead of batching
const useStreamingResults = () => {
  const [results, setResults] = useState<Result[]>([])

  const streamProcessor = useCallback((newResult: Result) => {
    setResults(prev => {
      // Upsert: update existing or add new
      const existingIndex = prev.findIndex(r => r.id === newResult.id)
      if (existingIndex >= 0) {
        const updated = [...prev]
        updated[existingIndex] = { ...updated[existingIndex], ...newResult }
        return updated
      }
      return [...prev, newResult].sort(byPriority)
    })
  }, [])

  return { results, streamProcessor }
}
```

### Speed Architecture Principles

**1. Instant First Impression**
- Show UI structure immediately (skeleton/optimistic)
- Never wait for all data before rendering
- Progressive enhancement as data streams in

**2. Fail Fast, Recover Faster**
- Timeout operations aggressively (2-5 seconds max)
- Circuit breakers prevent cascade failures
- Graceful degradation with partial results

**3. Parallelism + Partitioning**
- Group operations by service/endpoint for optimal batching
- Use structured concurrency for complex workflows
- Cancel slow operations that would block faster ones

**4. Memory-Speed Trade-offs**
- Fast gates (in-memory sets/maps) for instant checks
- LRU caches for recent operations
- Precomputed candidate generation

**5. Streaming Over Batching**
- Stream results as they resolve
- Update UI progressively
- Reconcile optimistic states when truth arrives

## NEXTMASTER ADVANCED PATTERNS

### 8. Partial Prerendering (PPR) - Next.js 15
```typescript
// app/layout.tsx - Enable PPR at layout level
export const experimental_ppr = true

// Static shell renders instantly, dynamic content streams in
const ProductPage = async ({ params }: { params: { id: string } }) => {
  return (
    <div>
      {/* Static shell - renders immediately */}
      <Header />
      <ProductNavigation />

      {/* Dynamic content - streams in */}
      <Suspense fallback={<Skeleton className="h-96" />}>
        <ProductDetails id={params.id} />
      </Suspense>

      {/* Static footer - renders immediately */}
      <Footer />
    </div>
  )
}

// Dynamic component that streams
async function ProductDetails({ id }: { id: string }) {
  const product = await fetchProduct(id) // This streams
  return <ProductCard product={product} />
}
```

### 9. Server Actions for Zero-Latency Mutations
```typescript
// app/actions/product.ts - Server Actions for instant mutations
'use server'

import { revalidatePath } from 'next/cache'

export async function updateProduct(formData: FormData) {
  const productId = formData.get('id') as string
  const name = formData.get('name') as string

  try {
    // Database mutation
    await db.update(products)
      .set({ name, updatedAt: new Date() })
      .where(eq(products.id, productId))

    // Revalidate specific paths only
    revalidatePath(`/products/${productId}`)
    revalidatePath('/products')

    return { success: true }
  } catch (error) {
    return { success: false, error: error.message }
  }
}

// Component using Server Action
const ProductForm = ({ product }: { product: Product }) => {
  return (
    <form action={updateProduct}>
      <input type="hidden" name="id" value={product.id} />
      <Input name="name" defaultValue={product.name} />
      <Button type="submit">Save Changes</Button>
    </form>
  )
}
```

### 10. In-Function Concurrency (50% Compute Reduction)
```typescript
// Concurrent data fetching within single function
async function getProductPageData(productId: string) {
  // Run all queries concurrently instead of sequentially
  const [product, reviews, relatedProducts, inventory] = await Promise.all([
    db.select().from(products).where(eq(products.id, productId)).limit(1),
    db.select().from(reviews).where(eq(reviews.productId, productId)).limit(10),
    db.select().from(products).where(ne(products.id, productId)).limit(5),
    db.select().from(inventory).where(eq(inventory.productId, productId)).limit(1)
  ])

  return {
    product: product[0],
    reviews,
    relatedProducts,
    inventory: inventory[0]
  }
}

// Use with React Suspense for streaming
const ProductPage = async ({ params }: { params: { id: string } }) => {
  const data = await getProductPageData(params.id)

  return (
    <div>
      <ProductDisplay product={data.product} inventory={data.inventory} />
      <Suspense fallback={<Skeleton className="h-48" />}>
        <ProductReviews reviews={data.reviews} />
      </Suspense>
      <Suspense fallback={<Skeleton className="h-64" />}>
        <RelatedProducts products={data.relatedProducts} />
      </Suspense>
    </div>
  )
}
```

### 11. Incremental Static Regeneration (ISR) Strategy
```typescript
// app/products/[id]/page.tsx - ISR without traditional caching
export const revalidate = 3600 // 1 hour

async function ProductPage({ params }: { params: { id: string } }) {
  const product = await fetchProduct(params.id)

  if (!product) {
    notFound()
  }

  return (
    <div>
      <h1>{product.name}</h1>
      <p>Last updated: {product.updatedAt}</p>
      {/* Static content that regenerates hourly */}
    </div>
  )
}

// On-demand revalidation via webhook
// app/api/revalidate/route.ts
export async function POST(request: Request) {
  const { path } = await request.json()

  try {
    await revalidatePath(path)
    return Response.json({ revalidated: true })
  } catch (err) {
    return Response.json({ error: 'Error revalidating' }, { status: 500 })
  }
}
```

### 12. Drizzle ORM Performance Patterns
```typescript
// lib/db/queries.ts - Optimized database patterns
import { eq, and, desc, count } from 'drizzle-orm'

// Prepared statements for repeated queries (50%+ faster)
const getProductById = db
  .select()
  .from(products)
  .where(eq(products.id, placeholder('id')))
  .prepare()

const getProductsWithPagination = db
  .select()
  .from(products)
  .limit(placeholder('limit'))
  .offset(placeholder('offset'))
  .prepare()

// Batch operations for bulk inserts
export async function createProductsBatch(productData: NewProduct[]) {
  return await db.insert(products).values(productData).returning()
}

// Optimized queries with joins and aggregations
export async function getProductsWithStats() {
  return await db
    .select({
      id: products.id,
      name: products.name,
      reviewCount: count(reviews.id),
      avgRating: avg(reviews.rating)
    })
    .from(products)
    .leftJoin(reviews, eq(products.id, reviews.productId))
    .groupBy(products.id)
    .orderBy(desc(count(reviews.id)))
}
```

### 13. Edge Computing + AI Batch Processing
```typescript
// app/api/generate-content/route.ts - Edge function for AI processing
export const runtime = 'edge'

import { openai } from '@ai-sdk/openai'
import { generateObject } from 'ai'

export async function POST(request: Request) {
  const { products } = await request.json()

  // Batch process multiple products
  const results = await Promise.allSettled(
    products.map(async (product: Product) => {
      const { object } = await generateObject({
        model: openai('gpt-4o-mini'),
        prompt: `Generate SEO-optimized content for: ${product.name}`,
        schema: z.object({
          title: z.string(),
          description: z.string(),
          keywords: z.array(z.string())
        })
      })
      return { productId: product.id, content: object }
    })
  )

  return Response.json({
    results: results.filter(r => r.status === 'fulfilled')
  })
}
```

### 14. Server Components with Streaming
```typescript
// Advanced Server Component patterns
async function ProductGrid({ category }: { category: string }) {
  // This runs on the server, data never sent to client
  const products = await getProductsByCategory(category)

  return (
    <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
      {products.map((product) => (
        <Suspense key={product.id} fallback={<ProductCardSkeleton />}>
          <ProductCard product={product} />
        </Suspense>
      ))}
    </div>
  )
}

// Client component only for interactivity
'use client'
function ProductCard({ product }: { product: Product }) {
  const [isLiked, setIsLiked] = useState(false)

  return (
    <Card className="group hover:shadow-lg transition-shadow">
      <CardContent className="p-4">
        <h3>{product.name}</h3>
        <p className="text-muted-foreground">{product.price}</p>
        <Button
          variant={isLiked ? "default" : "outline"}
          onClick={() => setIsLiked(!isLiked)}
        >
          {isLiked ? "❤️" : "🤍"}
        </Button>
      </CardContent>
    </Card>
  )
}
```

### 15. Image Optimization at Scale
```typescript
// lib/image-optimization.ts - Vercel Blob + optimization
import { put } from '@vercel/blob'
import sharp from 'sharp'

export async function optimizeAndUploadImage(
  file: File,
  productId: string
): Promise<string> {
  // Convert to WebP and optimize
  const buffer = Buffer.from(await file.arrayBuffer())
  const optimized = await sharp(buffer)
    .webp({ quality: 85 })
    .resize(800, 600, { fit: 'cover' })
    .toBuffer()

  // Upload to Vercel Blob
  const { url } = await put(`products/${productId}.webp`, optimized, {
    access: 'public',
    contentType: 'image/webp'
  })

  return url
}

// Component with optimized images
function ProductImage({ src, alt }: { src: string; alt: string }) {
  return (
    <Image
      src={src}
      alt={alt}
      width={800}
      height={600}
      priority // For above-fold images
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRgABAQAAAQ..." // Base64 blur
      className="object-cover rounded-lg"
    />
  )
}
```

### NextMaster Architecture Principles

**1. PPR + Streaming**
- Static shell renders instantly
- Dynamic content streams progressively
- Zero layout shift

**2. Server-First Architecture**
- Server Actions for mutations
- Server Components for data fetching
- Client components only for interactivity

**3. Database Optimization**
- Prepared statements for repeated queries
- Concurrent fetching within functions
- Batch operations for bulk updates

**4. Edge Computing**
- AI processing at the edge
- Image optimization pipeline
- Global distribution via Vercel Edge

**5. TypeScript-First (96.8%)**
- Strict type safety
- Runtime validation with Zod
- End-to-end type safety

### Mobile Responsiveness
- Use `/hooks/use-mobile.ts` for breakpoint detection
- Apply responsive patterns from shadcn examples
- Use shadcn responsive component variants
- **Test on real devices**: Always validate on actual mobile devices
- **Landscape/Portrait**: Support both orientations seamlessly

## Available shadcn/ui Components

**Core Components**: `Button`, `Input`, `Label`, `Textarea`, `Select`, `Checkbox`, `RadioGroup`, `Switch`
**Layout**: `Card`, `Sheet`, `Accordion`, `Collapsible`, `Separator`
**Navigation**: `NavigationMenu`, `Breadcrumb`, `Pagination`, `Command`, `Menubar`
**Data Display**: `Table`, `Badge`, `Avatar`, `Calendar`, `Progress`, `Skeleton`
**Feedback**: `Alert`, `Toast`, `Tooltip`, `HoverCard`, `Popover`
**Forms**: `Form`, `FormField`, `FormItem`, `FormLabel`, `FormControl`, `FormMessage`

## ACCESSIBILITY & PERFORMANCE MONITORING

### Web Vitals & Core Metrics
```typescript
// lib/analytics.ts - Performance monitoring
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals'

export function trackWebVitals() {
  getCLS(console.log) // Cumulative Layout Shift
  getFID(console.log) // First Input Delay
  getFCP(console.log) // First Contentful Paint
  getLCP(console.log) // Largest Contentful Paint
  getTTFB(console.log) // Time to First Byte
}

// app/layout.tsx - Monitor performance
useEffect(() => {
  if (process.env.NODE_ENV === 'production') {
    trackWebVitals()
  }
}, [])
```

### Accessibility Best Practices
```typescript
// Always include proper ARIA labels and semantic HTML
<Button
  aria-label="Save changes to profile"
  aria-describedby="save-help-text"
  disabled={isLoading}
>
  {isLoading ? "Saving..." : "Save Profile"}
</Button>

// Focus management for keyboard navigation
const FormModal = () => {
  const firstInputRef = useRef<HTMLInputElement>(null)

  useEffect(() => {
    firstInputRef.current?.focus()
  }, [])

  return (
    <Sheet>
      <SheetContent>
        <Input ref={firstInputRef} placeholder="Enter name" />
      </SheetContent>
    </Sheet>
  )
}
```

## ERROR HANDLING & RESILIENCE

### Error Boundaries with shadcn
```typescript
// components/error-boundary.tsx
'use client'

import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'
import { Button } from '@/components/ui/button'

export function ErrorBoundary({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="flex h-screen items-center justify-center p-4">
      <Alert variant="destructive" className="max-w-md">
        <AlertTitle>Something went wrong!</AlertTitle>
        <AlertDescription className="mt-2">
          {error.message || 'An unexpected error occurred.'}
        </AlertDescription>
        <Button
          variant="outline"
          onClick={reset}
          className="mt-4"
        >
          Try again
        </Button>
      </Alert>
    </div>
  )
}
```

### Graceful Degradation Patterns
```typescript
// components/feature-toggle.tsx - Progressive enhancement
const AdvancedFeature = ({ fallback }: { fallback: React.ReactNode }) => {
  const [isSupported, setIsSupported] = useState(false)

  useEffect(() => {
    // Check for feature support
    setIsSupported('IntersectionObserver' in window)
  }, [])

  if (!isSupported) {
    return <>{fallback}</>
  }

  return <IntersectionObserverComponent />
}

// Usage with shadcn fallback
<AdvancedFeature
  fallback={
    <Card>
      <CardContent>
        <Skeleton className="h-48 w-full" />
      </CardContent>
    </Card>
  }
/>
```

## DEPLOYMENT & MONITORING

### Environment Configuration
```typescript
// config/env.ts - Type-safe environment variables
import { z } from 'zod'

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  NEXTAUTH_SECRET: z.string().min(32),
  VERCEL_URL: z.string().optional(),
  NODE_ENV: z.enum(['development', 'production', 'test']),
})

export const env = envSchema.parse(process.env)
```

### Health Checks & Monitoring
```typescript
// app/api/health/route.ts - System health monitoring
export async function GET() {
  try {
    // Check database connection
    await db.select({ count: count() }).from(users).limit(1)

    // Check external services
    const externalCheck = await fetch('https://api.example.com/health', {
      signal: AbortSignal.timeout(5000)
    })

    return Response.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      services: {
        database: 'up',
        external: externalCheck.ok ? 'up' : 'down'
      }
    })
  } catch (error) {
    return Response.json({
      status: 'unhealthy',
      error: error.message
    }, { status: 500 })
  }
}
```

## SECURITY PATTERNS

### Input Validation & Sanitization
```typescript
// lib/validators.ts - Zod schemas for validation
export const createUserSchema = z.object({
  email: z.string().email().max(255),
  name: z.string().min(2).max(100).regex(/^[a-zA-Z\s]+$/),
  password: z.string().min(8).max(128)
})

// Rate limiting with Vercel Edge
// app/api/auth/login/route.ts
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(5, '10 m'),
})

export async function POST(request: Request) {
  const ip = request.headers.get('x-forwarded-for') ?? 'anonymous'
  const { success } = await ratelimit.limit(ip)

  if (!success) {
    return Response.json({ error: 'Too many requests' }, { status: 429 })
  }

  // Continue with login logic
}
```

## TESTING STRATEGY

### Component Testing Patterns
```typescript
// __tests__/components/button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react'
import { Button } from '@/components/ui/button'

describe('Button Component', () => {
  it('shows loading state correctly', () => {
    render(<Button loading>Save</Button>)

    expect(screen.getByRole('button')).toBeDisabled()
    expect(screen.getByText('Save')).toBeInTheDocument()
  })

  it('handles click events', () => {
    const handleClick = jest.fn()
    render(<Button onClick={handleClick}>Click me</Button>)

    fireEvent.click(screen.getByRole('button'))
    expect(handleClick).toHaveBeenCalledTimes(1)
  })
})
```

### E2E Testing with Playwright
```typescript
// tests/e2e/user-flow.spec.ts
import { test, expect } from '@playwright/test'

test('user can complete signup flow', async ({ page }) => {
  await page.goto('/signup')

  // Test mobile responsive design
  await page.setViewportSize({ width: 375, height: 667 })

  await page.fill('[data-testid="email-input"]', 'test@example.com')
  await page.fill('[data-testid="password-input"]', 'securepassword123')

  await page.click('[data-testid="submit-button"]')

  // Verify success state
  await expect(page.locator('[data-testid="success-message"]')).toBeVisible()

  // Check for performance
  const metrics = await page.evaluate(() => ({
    lcp: performance.getEntriesByType('largest-contentful-paint')[0]?.startTime,
    cls: performance.getEntriesByType('layout-shift').reduce((sum, entry) => sum + entry.value, 0)
  }))

  expect(metrics.lcp).toBeLessThan(2500) // LCP under 2.5s
  expect(metrics.cls).toBeLessThan(0.1)  // CLS under 0.1
})
```

## Important Notes

- **PRODUCTION APPLICATION**: Build real business features only - no demos, examples, or test routes
- **SHADCN ULTRATHINK**: All UI must use shadcn components, colors, and patterns
- **MINIMALISTIC DESIGN**: No borders, use `border-0`, subtle backgrounds (`bg-muted/20`, `bg-muted/30`)
- **ULTRA-COMPACT SIZING**: h-8 buttons, p-3/p-4 cards, w-64 sidebar, h-4 icons on desktop
- **ANTI-DUPLICATION ULTRATHINK**: NEVER create multiple variants - ONE header (DashboardHeader), ONE sidebar (SidebarDrill), aggressively delete unused components
- **MOBILE-FIRST ALWAYS**: Design for mobile, enhance for desktop - everything must feel like an app
- **PLAYWRIGHT HEADLESS**: Continuous testing and monitoring in headless mode for design validation
- **NO TEST ROUTES**: Never create test/demo routes - test everything through Playwright only
- **NO Dialogs or Tabs**: Avoid `Dialog`, `AlertDialog`, and `Tabs` components entirely
- **APP-LIKE EXPERIENCE**: Smooth, fast, responsive with touch-optimized interactions
- **SPEED OVER EVERYTHING**: Instant loading, < 150KB bundles, no large loading pages
- **COMPONENT LOADING ONLY**: Use stateful buttons and skeleton loading, never full-page loaders
- **ACCESSIBILITY FIRST**: WCAG 2.1 AA compliance, keyboard navigation, screen reader support
- **ERROR RESILIENCE**: Graceful degradation, circuit breakers, proper error boundaries
- **TYPE-SAFE EVERYTHING**: Runtime validation with Zod, end-to-end type safety
- **PERFORMANCE MONITORING**: Track Core Web Vitals, LCP < 2.5s, CLS < 0.1
- **SECURITY BY DESIGN**: Input validation, rate limiting, proper authentication
- **PLAYWRIGHT TESTING ONLY**: All feature testing through Playwright headless mode, no test routes
- **BUSINESS VALUE REQUIRED**: Every feature must provide clear business or user value
- **Turbopack enabled**: All dev/build commands use `--turbopack` flag
- **No custom styling**: Use shadcn variants and Tailwind utilities only
- **Complete UI library**: 41 shadcn/ui components ready for immediate use (excluding Dialog/Tabs)
- **Real device testing**: Always validate on actual mobile devices
- **Cross-browser compatibility**: Test across different browsers and screen sizes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/byronwade)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/byronwade)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
