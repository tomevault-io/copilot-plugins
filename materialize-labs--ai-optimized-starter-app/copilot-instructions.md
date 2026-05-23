## frontend

> Follow these rules when working on the frontend.

# Frontend Architecture Guidelines

## Technology Stack

Our frontend leverages a modern stack designed for performance, developer experience, and maintainability:

- **Framework**: Next.js 15+ with App Router
- **Styling**: Tailwind CSS for utility-first styling
- **UI Components**: Shadcn UI as a component foundation
- **Animations**: Framer Motion for fluid animations
- **Icons**: Lucide React for consistent iconography

## Core Principles

1. **Type Safety**: Leverage TypeScript for robust, maintainable code
2. **Component-Driven Development**: Build reusable, composable components 
3. **Performance-First**: Optimize for core web vitals and user experience
4. **Accessibility**: Ensure WCAG compliance in all UIs
5. **Progressive Enhancement**: Build resilient interfaces that work across devices

## Component Architecture

### Component Types

Our application uses both **Server Components** and **Client Components** following Next.js paradigms:

| Component Type | Use Case | Key Considerations |
|---------------|----------|-------------------|
| **Server Components** | Data fetching, SEO-critical content | No `useState`, no event handlers |
| **Client Components** | Interactive UIs, forms, animations | Add `"use client"` directive at top of file |

### Component Organization

```
app/
├── [route]/              # Route directories
│   ├── _components/      # Route-specific components
│   ├── page.tsx          # Route page component
│   └── layout.tsx        # Route layout component
components/
├── ui/                   # Shared UI components
└── [feature]/            # Feature-specific shared components
```

#### Naming Conventions

- Use kebab-case for all component files: `data-table.tsx` 
- Use PascalCase for component names: `DataTable`
- Group related components in feature-specific directories
- Suffix context providers with `Provider`: `AuthProvider`

### Component Guidelines

#### Server vs Client Components

- Always add the appropriate directive at the top of your file:
  - `"use server"` for server components
  - `"use client"` for client components

- Server components should:
  - Perform data fetching
  - Pass data to client components via props
  - Minimize client-side JavaScript
  
- Client components should:
  - Handle user interactions
  - Manage local state
  - Implement animations and effects

#### Code Structure

- Import ordering:
  1. React and Next.js imports
  2. Third-party libraries
  3. Internal components and utilities
  4. Types and styles

- Export components as named exports for components meant to be used within a directory, and as default exports for page components or major feature components

## Server Component Patterns

### Data Fetching

- Fetch data directly in server components using appropriate patterns:

```tsx
// Good: Fetch in server components
async function ProductList() {
  const products = await getProductsAction()

  return <ProductGrid products={products.data} />
}
```

- Use Suspense boundaries for asynchronous content:

```tsx
// page.tsx
export default function Page() {
  return (
    <div>
      <Header />
      <Suspense fallback={<ProductsSkeleton />}>
        <ProductsContent />
      </Suspense>
    </div>
  )
}

// Async component in same file or imported
async function ProductsContent() {
  const products = await getProducts()

  return <ProductList products={products.data} />
}
```

### Detailed Examples

#### Server Layout Example

```tsx
// app/dashboard/layout.tsx
import { SidebarNav } from "./_components/sidebar-nav"
import { DashboardHeader } from "./_components/dashboard-header"
import { getProfile } from "@/actions/profile"
import { redirect } from "next/navigation"

export default async function DashboardLayout({
  children
}: {
  children: React.ReactNode
}) {
  const profile = await getProfile()
  
  if (!profile.isSuccess) {
    return redirect("/login")
  }
  
  return (
    <div className="flex min-h-screen flex-col">
      <DashboardHeader user={profile.data} />
      
      <div className="flex flex-1">
        <SidebarNav />
        <main className="flex-1 p-6">
          {children}
        </main>
      </div>
    </div>
  )
}
```

#### Server Page with Suspense

```tsx
// app/dashboard/page.tsx
import { Suspense } from "react"
import { DashboardSkeleton } from "./_components/dashboard-skeleton"
import { DashboardMetrics } from "./_components/dashboard-metrics"
import { DashboardCharts } from "./_components/dashboard-charts"

export default function DashboardPage() {
  return (
    <div className="space-y-6">
      <h1 className="text-3xl font-bold">Dashboard</h1>
      
      <Suspense fallback={<DashboardSkeleton type="metrics" />}>
        <DashboardMetricsContent />
      </Suspense>
      
      <Suspense fallback={<DashboardSkeleton type="charts" />}>
        <DashboardChartsContent />
      </Suspense>
    </div>
  )
}

async function DashboardMetricsContent() {
  const metrics = await getMetrics()

  return <DashboardMetrics data={metrics.data} />
}

async function DashboardChartsContent() {
  const chartData = await getChartData()

  return <DashboardCharts data={chartData.data} />
}
```

## Client Component Patterns

### State Management

- Use React's built-in state management for component-level state
- For more complex state, consider React Context or state management libraries
- Pass initial data from server components to hydrate client components

```tsx
// _components/data-table.tsx
"use client"

import { useState } from "react"
import { Button } from "@/components/ui/button"
import type { Product } from "@/types"

interface DataTableProps {
  initialData: Product[]
}

export function DataTable({ initialData }: DataTableProps) {
  const [data, setData] = useState(initialData)
  const [sortBy, setSortBy] = useState<keyof Product | null>(null)
  
  // Component logic...
  
  return (
    <div>
      {/* Table implementation */}
    </div>
  )
}
```

### Form Handling

- Use controlled components for forms
- Implement proper form validation
- Use server actions for form submissions

```tsx
"use client"

import { useState } from "react"
import { z } from "zod"
import { Button } from "@/components/ui/button"
import { Input } from "@/components/ui/input"
import { createContact } from "@/actions/db/contacts"

const formSchema = z.object({
  name: z.string().min(1, "Name is required"),
  email: z.string().email("Invalid email format").optional().nullable()
})

export function ContactForm() {
  const [name, setName] = useState("")
  const [email, setEmail] = useState("")
  const [errors, setErrors] = useState<Record<string, string>>({})
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault()
    
    try {
      // Validate form
      formSchema.parse({ name, email })
      
      // Submit form using server action
      await createContact({ name, email })
      
      // Reset form
      setName("")
      setEmail("")
      setErrors({})
    } catch (error) {
      if (error instanceof z.ZodError) {
        // Transform Zod errors into a usable format
        const newErrors: Record<string, string> = {}
        error.errors.forEach((err) => {
          if (err.path[0]) {
            newErrors[err.path[0] as string] = err.message
          }
        })
        setErrors(newErrors)
      }
    }
  }
  
  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <Input 
          value={name}
          onChange={(e) => setName(e.target.value)}
          placeholder="Name"
        />
        {errors.name && <p className="text-red-500 text-sm mt-1">{errors.name}</p>}
      </div>
      
      <div>
        <Input 
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          placeholder="Email"
        />
        {errors.email && <p className="text-red-500 text-sm mt-1">{errors.email}</p>}
      </div>
      
      <Button type="submit">Submit</Button>
    </form>
  )
}
```

## Styling Guidelines

### Tailwind CSS Usage

- Use Tailwind utility classes for styling
- Follow a consistent order of utility classes:
  1. Layout (display, position)
  2. Box model (width, height, margin, padding)
  3. Typography
  4. Visual (colors, background, etc.)
  5. Miscellaneous

```tsx
// Recommended class ordering
<div className="
  flex justify-between items-center 
  w-full h-16 px-4 
  text-sm font-medium 
  bg-white dark:bg-gray-900 border-b
  transition-colors duration-200
">
```

- Extract common patterns into component classes with `cn` utility:

```tsx
import { cn } from "@/lib/utils"

export function Card({ className, ...props }: React.HTMLAttributes<HTMLDivElement>) {
  return (
    <div 
      className={cn(
        "rounded-lg border bg-card p-4 shadow-sm", 
        className
      )} 
      {...props} 
    />
  )
}
```

### Theme Consistency

- Use design tokens from your theme for spacing, color, etc. instead of arbitrary values
- For colors, use semantic color names from your theme (primary, secondary, etc.)
- Maintain consistent spacing using Tailwind's spacing scale

## Animation Guidelines

### Framer Motion Best Practices

- Keep animations subtle and purposeful
- Use motion variants for coordinated animations
- Implement animations that respond to user interaction
- Consider reduced motion preferences

```tsx
"use client"

import { motion } from "framer-motion"

const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1
    }
  }
}

const itemVariants = {
  hidden: { y: 20, opacity: 0 },
  visible: {
    y: 0,
    opacity: 1,
    transition: {
      type: "spring",
      stiffness: 300,
      damping: 24
    }
  }
}

export function AnimatedList({ items }) {
  return (
    <motion.ul
      variants={containerVariants}
      initial="hidden"
      animate="visible"
      className="space-y-4"
    >
      {items.map((item) => (
        <motion.li
          key={item.id}
          variants={itemVariants}
          className="rounded-md border p-4"
        >
          {item.content}
        </motion.li>
      ))}
    </motion.ul>
  )
}
```

## Performance Optimization

### Image Optimization

- Use Next.js Image component for all images
- Specify proper width and height attributes
- Use responsive sizes for different viewports
- Implement lazy loading for below-the-fold images

```tsx
import Image from "next/image"

export function ProfileCard({ user }) {
  return (
    <div className="flex items-center space-x-4">
      <Image
        src={user.avatarUrl}
        alt={`${user.name}'s profile picture`}
        width={64}
        height={64}
        className="rounded-full"
        priority={false}
      />
      <div>
        <h3 className="font-medium">{user.name}</h3>
        <p className="text-sm text-gray-500">{user.title}</p>
      </div>
    </div>
  )
}
```

### Component Memoization

- Memoize expensive computations and renders with `useMemo` and `memo`
- Use callback references with `useCallback` for event handlers passed to child components
- Implement proper dependency arrays in hooks

## Accessibility Guidelines

- Use semantic HTML elements when possible
- Include proper ARIA attributes when needed
- Ensure sufficient color contrast
- Implement keyboard navigation
- Test with screen readers

```tsx
"use client"

import { useState } from "react"
import { Button } from "@/components/ui/button"

export function Disclosure({ title, children }) {
  const [isOpen, setIsOpen] = useState(false)
  
  return (
    <div className="border rounded-md">
      <Button
        type="button"
        onClick={() => setIsOpen(!isOpen)}
        aria-expanded={isOpen}
        className="flex w-full items-center justify-between p-4"
      >
        <span>{title}</span>
        <span className="text-xs" aria-hidden="true">
          {isOpen ? "−" : "+"}
        </span>
      </Button>
      
      {isOpen && (
        <div className="p-4 pt-0">
          {children}
        </div>
      )}
    </div>
  )
}
```

---
> Source: [materialize-labs/ai-optimized-starter-app](https://github.com/materialize-labs/ai-optimized-starter-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
