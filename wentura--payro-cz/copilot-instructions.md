## payro-cz

> - **DEFAULT**: Always start with Server Components

# Cursor AI Rules - Next.js Performance & Best Practices

## 🎯 Core Principles

### 1. Server Components First
- **DEFAULT**: Always start with Server Components
- **RULE**: Add `"use client"` ONLY when absolutely necessary (interactivity, hooks, browser APIs)
- **PATTERN**: Fetch data in Server Components, pass as props to Client Components
- **AVOID**: Client Components that fetch data in `useEffect` - fetch on server instead

### 2. Performance First
- **CACHING**: Use Next.js `unstable_cache` for static/reference data
- **PARALLEL FETCHING**: Use `Promise.all()` instead of sequential awaits
- **DYNAMIC IMPORTS**: Lazy load modals, heavy components, rarely-used features
- **BUNDLE SIZE**: Monitor and optimize - use dynamic imports for code splitting
- **DATABASE**: Add composite indexes for common query patterns

### 3. Minimalist Architecture
- **KEEP IT SIMPLE**: Avoid over-engineering
- **YAGNI**: You Aren't Gonna Need It - don't add features until needed
- **DRY**: Don't Repeat Yourself, but don't abstract prematurely
- **FOCUS**: One component = one responsibility

### 4. Full Functionality
- **COMPLETE FEATURES**: Don't ship half-baked features
- **ERROR HANDLING**: Always handle errors gracefully
- **LOADING STATES**: Show loading indicators for async operations
- **VALIDATION**: Validate on both client and server
- **ACCESSIBILITY**: Proper ARIA labels, keyboard navigation

## 📐 Architecture Patterns

### Server Component Pattern
```javascript
// ✅ GOOD: Server Component fetching data
export default async function Page() {
  const data = await fetchData();
  return <ClientForm data={data} />;
}

// ❌ BAD: Client Component fetching data
"use client";
export default function Page() {
  const [data, setData] = useState(null);
  useEffect(() => {
    fetchData().then(setData);
  }, []);
  return <Form data={data} />;
}
```

### Parallel Data Fetching
```javascript
// ✅ GOOD: Parallel fetching
const [data1, data2, data3] = await Promise.all([
  fetchData1(),
  fetchData2(),
  fetchData3(),
]);

// ❌ BAD: Sequential fetching (waterfall)
const data1 = await fetchData1();
const data2 = await fetchData2();
const data3 = await fetchData3();
```

### Dynamic Imports
```javascript
// ✅ GOOD: Lazy load heavy components
const HeavyModal = dynamic(() => import('./HeavyModal'), {
  ssr: false,
  loading: () => null,
});

// ❌ BAD: Import everything upfront
import HeavyModal from './HeavyModal';
```

## 🗂️ File Organization

### Naming Conventions
- **Files**: `kebab-case.js` (e.g., `new-invoice-form.js`)
- **Components**: `PascalCase` (e.g., `NewInvoiceForm`)
- **Utilities**: `camelCase` (e.g., `formatCurrency`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `MAX_INVOICE_ITEMS`)

### Directory Structure
```
app/
├── (route-groups)/    # Route groups for layouts
├── [dynamic]/         # Dynamic routes
├── api/               # API routes
├── components/        # Reusable components
│   ├── ui/           # Basic UI components
│   └── ...           # Feature components
└── lib/              # Utilities and services
    └── services/     # Data fetching services
```

## 🔒 Security & Data

### Multi-tenancy
- **ALWAYS**: Scope queries by `user_id`
- **NEVER**: Trust client-side user_id
- **VALIDATE**: User ownership on server side
- **PATTERN**: `WHERE user_id = $1 AND ...`

### Authentication
- **SERVER-SIDE**: Always verify auth on server
- **COOKIES**: Use HTTP-only cookies for sessions
- **PASSWORDS**: Hash with bcrypt (10+ rounds minimum)
- **TOKENS**: Expire tokens, validate on server

### Validation
- **ZOD**: Use Zod schemas for validation
- **BOTH**: Validate on client AND server
- **SANITIZE**: Sanitize user input
- **ESCAPE**: Escape output to prevent XSS

## ⚡ Performance Rules

### Caching Strategy
```javascript
// Reference data (static, rarely changes)
export const getReferenceData = unstable_cache(
  async () => fetchFromDB(),
  ['reference-data'],
  { revalidate: 3600, tags: ['reference-data'] }
);

// User-specific data (dynamic)
export async function getUserData(userId) {
  // No cache, always fresh
  return await fetchUserData(userId);
}
```

### Database Optimization
- **INDEXES**: Add composite indexes for common queries
- **PARTIAL INDEXES**: Use `WHERE` clauses in indexes when possible
- **QUERY PATTERNS**: Optimize for actual query patterns, not theoretical
- **ANALYZE**: Run `ANALYZE` after adding indexes

### Bundle Optimization
- **ANALYZE**: Use `@next/bundle-analyzer` regularly
- **SPLIT**: Split large components into smaller chunks
- **LAZY**: Lazy load routes, modals, heavy features
- **TREE-SHAKE**: Use ES modules, avoid default exports for utilities

## 🎨 Code Quality

### Component Design
- **SMALL**: Keep components small and focused
- **PROPS**: Prefer props over context when possible
- **COMPOSITION**: Compose small components into larger ones
- **REUSABLE**: Make components reusable, not too specific

### Error Handling
```javascript
// ✅ GOOD: Proper error handling
try {
  const data = await fetchData();
  return <Success data={data} />;
} catch (error) {
  console.error('Error:', error);
  return <Error message="Something went wrong" />;
}

// ❌ BAD: Silent failures
const data = await fetchData(); // What if this fails?
return <Component data={data} />;
```

### Loading States
```javascript
// ✅ GOOD: Show loading state
export default async function Page() {
  const data = await fetchData();
  return <Content data={data} />;
}

// With Suspense boundary in parent
<Suspense fallback={<Loading />}>
  <Page />
</Suspense>
```

## 📝 Documentation

### Code Comments
- **WHY**: Explain why, not what
- **COMPLEX**: Comment complex business logic
- **TODOs**: Use TODO comments for future improvements
- **AVOID**: Don't comment obvious code

### README
- **SETUP**: Clear setup instructions
- **ENV**: Document all environment variables
- **ARCHITECTURE**: Explain key architectural decisions
- **DEPLOYMENT**: Deployment instructions

## 🚀 Deployment

### Environment Variables
- **DOCUMENT**: All env vars in README
- **VALIDATE**: Validate required env vars on startup
- **SECRETS**: Never commit secrets to git
- **TYPES**: Use TypeScript or JSDoc for env var types

### Build Optimization
- **STATIC**: Use static generation when possible
- **ISR**: Use ISR for semi-static content
- **SSR**: Use SSR only when necessary
- **MONITOR**: Monitor bundle size and build time

## ✅ Checklist for New Features

Before implementing a new feature:

- [ ] Is this a Server Component? (default yes)
- [ ] Can data be fetched in parallel?
- [ ] Is caching appropriate?
- [ ] Are database indexes optimized?
- [ ] Is error handling implemented?
- [ ] Are loading states shown?
- [ ] Is validation on both client and server?
- [ ] Is the component accessible?
- [ ] Is the code documented?
- [ ] Is the bundle size acceptable?

## 🎯 Success Metrics

A well-optimized Next.js app should have:

- **FCP**: First Contentful Paint < 1s
- **TTI**: Time to Interactive < 2s
- **Bundle**: Initial bundle < 100KB (gzipped)
- **LCP**: Largest Contentful Paint < 2.5s
- **CLS**: Cumulative Layout Shift < 0.1

---

**Remember**: Performance, simplicity, and full functionality are not optional - they are requirements.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wentura) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
