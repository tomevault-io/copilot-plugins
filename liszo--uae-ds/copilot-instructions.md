## uae-ds

> Next.js 14/15 agency website showcasing digital solutions, services, and portfolio.

# UAE Digital Solution Agency - Next.js Frontend Rules

## Project Overview
Next.js 14/15 agency website showcasing digital solutions, services, and portfolio.
Frontend-only project connecting to headless WordPress backend via GraphQL.

## Tech Stack
- Next.js 14/15 (App Router)
- React 18+ with TypeScript
- Tailwind CSS 4.x
- Framer Motion
- React Hook Form + Zod
- WPGraphQL client

## Coding Standards

### Next.js App Router
- Server Components by default
- 'use client' only for: hooks, events, browser APIs, animations
- async/await for data fetching in Server Components
- generateMetadata() for all SEO
- generateStaticParams() for dynamic routes
- Route groups for organization: (marketing), etc.

### Component Patterns
```typescript
// Server Component (default pattern)
export async function CasesGrid() {
  const cases = await getCases()
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
      {cases.map(c => <CaseCard key={c.id} {...c} />)}
    </div>
  )
}

// Client Component (when needed)
'use client'
import { motion } from 'framer-motion'

export function AnimatedSection({ children }) {
  return (
    <motion.div
      initial={{ opacity: 0, y: 24 }}
      whileInView={{ opacity: 1, y: 0 }}
      viewport={{ once: true }}
    >
      {children}
    </motion.div>
  )
}
```

### TypeScript
- Strict mode enabled
- No 'any' without justification
- Interface for props, Type for unions/intersections
- Explicit return types for exported functions
```typescript
interface CaseCardProps {
  title: string
  client: string
  image: string
  tags: string[]
  href: string
}

export function CaseCard({ title, client, image, tags, href }: CaseCardProps): JSX.Element {
  // Implementation
}
```

### Tailwind CSS
- Mobile-first responsive design
- Semantic class grouping: layout → spacing → colors → typography → effects
- Use cn() utility for conditional classes
- Extract variants with CVA for complex components
```typescript
import { cva } from 'class-variance-authority'

const buttonVariants = cva(
  'inline-flex items-center justify-center rounded-lg transition',
  {
    variants: {
      variant: {
        primary: 'bg-blue-600 text-white hover:bg-blue-700',
        secondary: 'bg-gray-200 text-gray-900 hover:bg-gray-300',
      },
      size: {
        sm: 'h-9 px-3 text-sm',
        md: 'h-11 px-5',
        lg: 'h-13 px-7 text-lg',
      },
    },
  }
)
```

### Styling System
- Colors: Primary brand blue, neutral grays, semantic colors
- Spacing: Tailwind's 4px scale
- Typography: text-base (16px) body, text-6xl+ for heroes
- Border radius: rounded-lg default, rounded-2xl for cards
- Container: max-w-7xl mx-auto px-6 lg:px-8

### Image Optimization
- ALWAYS use next/image, never <img>
- priority={true} for above-fold images
- fill with object-cover for backgrounds
- Specify width/height for layout shift prevention
```typescript
<Image
  src={image}
  alt={title}
  width={1200}
  height={800}
  priority={isAboveFold}
  className="rounded-lg"
/>
```

### Animation Standards
- Framer Motion for all animations
- useReducedMotion support required
- Entrance animations: fade + subtle translate (y: 24)
- Duration: 0.3-0.6s for most animations
- viewport={{ once: true }} for scroll triggers
```typescript
<motion.div
  initial={{ opacity: 0, y: 24 }}
  whileInView={{ opacity: 1, y: 0 }}
  viewport={{ once: true, margin: '-100px' }}
  transition={{ duration: 0.5 }}
>
  {children}
</motion.div>
```

### Form Handling
- React Hook Form for state management
- Zod for schema validation
- Server Actions for submission
```typescript
const formSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  message: z.string().min(10),
})

type FormData = z.infer<typeof formSchema>

export function ContactForm() {
  const { register, handleSubmit, formState: { errors } } = useForm<FormData>({
    resolver: zodResolver(formSchema)
  })
  // Implementation
}
```

### Data Fetching (WordPress)
- Server Components fetch directly
- ISR with revalidate: 3600 for content pages
- Cache WordPress GraphQL responses
```typescript
export async function getCases() {
  const res = await fetch(process.env.WORDPRESS_GRAPHQL_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ query: CASES_QUERY }),
    next: { revalidate: 3600 }
  })
  return res.json()
}
```

### SEO Implementation
- Metadata API for all pages
- Structured data (JSON-LD)
- sitemap.ts and robots.ts
- Open Graph and Twitter cards
```typescript
export const metadata: Metadata = {
  title: 'Page Title - UAE Digital Solution',
  description: 'Compelling description for SEO',
  openGraph: {
    title: 'OG Title',
    description: 'OG Description',
    images: ['/og-image.jpg'],
  },
}
```

### Performance Rules
- Keep bundle size minimal
- Dynamic imports for heavy components
- Lazy load below-fold content
- Optimize images (WebP, proper sizing)
- Minimize 'use client' boundaries
- ISR for static-ish content
- Monitor Core Web Vitals

### Accessibility (WCAG 2.1 AA)
- Semantic HTML (nav, main, article, section)
- Proper heading hierarchy
- Alt text for images
- ARIA labels where semantic HTML insufficient
- Keyboard navigation
- Focus visible states
- Color contrast 4.5:1+

### Error Handling
- error.tsx for route errors
- Loading.tsx for suspense fallbacks
- Try-catch in Server Components
- Proper error messages (user-friendly)
- Fallback UI for failed data fetches

### Naming Conventions
- Components: PascalCase (CaseCard.tsx)
- Functions: camelCase (getCases)
- Constants: UPPER_SNAKE_CASE (API_URL)
- Files/folders: kebab-case (case-studies/)
- CSS classes: Tailwind utilities

### Import Order
```typescript
// 1. External dependencies
import { Metadata } from 'next'
import { motion } from 'framer-motion'

// 2. Internal absolute imports
import { Button } from '@/components/ui/Button'
import { getCases } from '@/lib/wordpress'

// 3. Relative imports
import { CaseCard } from './CaseCard'
import { formatDate } from '../utils'

// 4. Types
import type { Case } from '@/types/case'
```

## AI Behavior Guidelines

### Before Coding
1. Read existing patterns in similar components
2. Check current implementations before creating new ones
3. Understand the full context of the request
4. Ask clarifying questions if ambiguous

### When Creating Components
1. Check if similar component exists
2. Server Component first, Client only if necessary
3. Add proper TypeScript types
4. Include responsive design
5. Add loading/error states
6. Consider accessibility
7. Follow design system

### When Modifying Code
1. Preserve existing functionality
2. Match current code style
3. Update related documentation
4. Consider side effects
5. Test edge cases mentally

### When Fixing Bugs
1. Identify root cause
2. Explain the issue
3. Provide comprehensive fix
4. Check for similar issues
5. Prevent future occurrences

### Performance Optimization
1. Measure before optimizing
2. Focus on actual bottlenecks
3. Safe optimizations only
4. Verify improvements
5. Document trade-offs

## Common Patterns

### Page Structure
```typescript
// app/(marketing)/services/page.tsx
import { Metadata } from 'next'
import { Hero } from '@/components/sections/Hero'
import { Services } from '@/components/sections/Services'

export const metadata: Metadata = {
  title: 'Services - UAE Digital Solution',
  description: 'Our comprehensive digital services',
}

export default function ServicesPage() {
  return (
    <>
      <Hero
        title="Our Services"
        subtitle="Comprehensive digital solutions"
      />
      <Services />
    </>
  )
}
```

### Hero Component Pattern
```typescript
export function Hero({ title, subtitle, cta, background }) {
  return (
    <section className="relative flex h-screen items-center justify-center">
      {background && (
        <Image
          src={background}
          alt=""
          fill
          className="object-cover brightness-50"
          priority
        />
      )}
      <Container className="relative z-10 text-center">
        <h1 className="text-6xl font-bold tracking-tight lg:text-8xl">
          {title}
        </h1>
        <p className="mt-6 text-xl lg:text-2xl">{subtitle}</p>
        {cta && <Button size="lg" className="mt-10">{cta}</Button>}
      </Container>
    </section>
  )
}
```

### Card Component Pattern
```typescript
export function CaseCard({ title, client, image, tags, href }: CaseCardProps) {
  return (
    <Link href={href}>
      <div className="group overflow-hidden rounded-2xl">
        <div className="relative aspect-[4/3] overflow-hidden">
          <Image
            src={image}
            alt={title}
            fill
            className="object-cover transition duration-500 group-hover:scale-105"
          />
        </div>
        <div className="p-6">
          <p className="mb-2 text-sm text-gray-500">{client}</p>
          <h3 className="mb-3 text-xl font-bold">{title}</h3>
          <div className="flex flex-wrap gap-2">
            {tags.map(tag => (
              <Badge key={tag}>{tag}</Badge>
            ))}
          </div>
        </div>
      </div>
    </Link>
  )
}
```

## Testing Guidelines
- Unit tests for utilities
- Component tests with React Testing Library
- E2E tests for critical flows
- Test responsive behavior
- Test accessibility with axe

## Git Workflow
- Feature branches from main
- Descriptive commit messages
- PR reviews required
- Squash merge to main

## Documentation
- JSDoc for complex functions
- README for setup instructions
- Inline comments for non-obvious logic
- Type definitions self-document

## Don't
- ❌ Use Pages Router patterns
- ❌ Create Client Components unnecessarily
- ❌ Use <img> instead of next/image
- ❌ Ignore TypeScript errors
- ❌ Skip responsive design
- ❌ Forget accessibility
- ❌ Premature optimization
- ❌ Break existing patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/liszo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
