## songbird

> This is a Next.js 14+ application for tracking "Song of the Day" entries with social features, analytics, and music discovery.

# Claude Code Rules for SongBird (SOTD) Project

## Project Overview
This is a Next.js 14+ application for tracking "Song of the Day" entries with social features, analytics, and music discovery.

## Tech Stack
- **Framework**: Next.js 14+ (App Router)
- **Language**: TypeScript
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: NextAuth.js
- **Styling**: Tailwind CSS
- **Deployment**: Vercel

## Code Style & Conventions

### TypeScript
- Use strict TypeScript typing - avoid `any` when possible
- Define interfaces for all data structures
- Use type inference where appropriate but be explicit for function parameters and returns

### React/Next.js Patterns
- Use functional components with hooks
- Prefer `'use client'` directive for client components
- Use Server Components by default (no directive)
- Always handle loading states before showing "no data" messages
- Use proper error boundaries and error handling

### File Structure
- API routes: `app/api/[route]/route.ts`
- Components: `components/[ComponentName].tsx`
- Pages: `app/[route]/page.tsx`
- Utilities: `lib/[utility].ts`
- Database schema: `prisma/schema.prisma`

## Database Patterns
- Use Prisma for all database operations
- Always use `select` or `include` to limit fields (never fetch all fields)
- Use indexes for frequently queried fields (userId, date combinations)
- Handle connection pooling properly for Vercel serverless
- Never fetch base64 images in bulk queries (use `excludeImages=true`)

## API Route Patterns
- Always check authentication with `getServerSession(authOptions)`
- Return proper HTTP status codes
- Use NextResponse.json for responses
- Handle errors gracefully with try/catch
- Limit response sizes for performance

## Performance Guidelines
- Always show loading states instead of "no data" while fetching
- Use pagination for large datasets (pageSize: 100-1000 depending on data)
- Avoid unnecessary count queries - use length checks instead
- Optimize images (Next.js Image component, exclude when not needed)
- Use database indexes for common query patterns

## Component Patterns
- Loading states should appear BEFORE empty states
- Check `loading` state before checking data length
- Use consistent loading messages: "Loading...", "Loading [resource]..."
- Empty states should only show when `!loading && data.length === 0`

## UI/UX Guidelines
- Use consistent color scheme: `bg-bg`, `text-text`, `bg-surface`, `text-accent`
- Loading indicators: Use `animate-pulse` or loading spinners
- Error messages: Clear, user-friendly, with actionable suggestions
- Responsive design: Mobile-first, use `sm:`, `md:` breakpoints

## Common Patterns

### Entry Fetching
```typescript
const [entries, setEntries] = useState<Entry[]>([])
const [loading, setLoading] = useState(false)

const fetchEntries = async () => {
  setLoading(true)
  try {
    const res = await fetch('/api/entries')
    const data = await res.json()
    if (res.ok) {
      setEntries(data.entries)
    }
  } catch (error) {
    console.error('Error:', error)
  } finally {
    setLoading(false)
  }
}
```

### Loading State Pattern
```typescript
{loading ? (
  <div>Loading...</div>
) : entries.length > 0 ? (
  <div>{/* Show entries */}</div>
) : (
  <div>No entries found</div>
)}
```

## Testing & Deployment
- Test locally before deploying
- Check Vercel logs for serverless function issues
- Database migrations: Use `prisma migrate dev` for local, `prisma db push` for Vercel
- Environment variables: Set in Vercel dashboard, use `.env.local` for local dev

## Code Quality
- Keep functions focused and single-purpose
- Extract reusable logic into utility functions
- Comment complex business logic
- Use meaningful variable and function names
- Handle edge cases (empty arrays, null values, errors)

---
> Source: [diiimodiiimo/Songbird](https://github.com/diiimodiiimo/Songbird) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
