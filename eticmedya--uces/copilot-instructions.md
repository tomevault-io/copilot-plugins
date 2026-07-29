## uces

> > Production-grade standards for professional development

# UCES Code Conventions

> Production-grade standards for professional development

---

## 1. TypeScript Standards

### Strict Configuration
```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

### Type Requirements
```typescript
// REQUIRED: Explicit function signatures
function processUser(user: User): ProcessedUser { }

// REQUIRED: Interface over type for objects
interface UserProfile {
  id: string
  name: string
  email: string
}

// REQUIRED: Discriminated unions for variants
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string }
```

### Prohibited
```typescript
// BLOCKED: any type
const data: any = fetch()  // NO

// BLOCKED: Non-null assertion without check
user!.name  // NO

// BLOCKED: Type assertion without validation
data as User  // NO - use type guards
```

---

## 2. React Patterns

### Component Structure
```typescript
// 1. Imports (external, internal, types)
// 2. Types/Interfaces
// 3. Component
// 4. Styles (if colocated)

import { useState } from 'react'
import { Button } from '@/components/ui'
import type { UserProps } from './types'

interface Props extends UserProps {
  onUpdate: (id: string) => void
}

export function UserCard({ user, onUpdate }: Props) {
  // hooks first
  const [editing, setEditing] = useState(false)

  // handlers
  const handleSave = () => { }

  // render
  return (...)
}
```

### State Management Rules
```
Server data     → TanStack Query (useQuery/useMutation)
Global UI state → Zustand
Local UI state  → useState/useReducer
Form state      → React Hook Form
URL state       → nuqs or useSearchParams
```

### Mandatory State Handling
```typescript
export function DataList() {
  const { data, isLoading, error, refetch } = useQuery(...)

  // ALL states required:
  if (isLoading) return <Skeleton />
  if (error) return <Error onRetry={refetch} />
  if (!data?.length) return <Empty onCreate={...} />

  return <List items={data} />
}
```

---

## 3. API Design

### Endpoint Structure
```typescript
// app/api/resources/route.ts

export async function GET(request: Request) {
  // 1. Auth
  const session = await auth()
  if (!session) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // 2. Query params
  const { searchParams } = new URL(request.url)
  const page = parseInt(searchParams.get('page') ?? '1')

  // 3. Execute
  const data = await db.resource.findMany({
    where: { userId: session.user.id },
    skip: (page - 1) * 20,
    take: 20,
  })

  return Response.json(data)
}

export async function POST(request: Request) {
  // 1. Auth
  const session = await auth()
  if (!session) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 })
  }

  // 2. Validate
  const body = await request.json()
  const parsed = CreateSchema.safeParse(body)
  if (!parsed.success) {
    return Response.json({ error: parsed.error.flatten() }, { status: 400 })
  }

  // 3. Execute
  try {
    const created = await db.resource.create({
      data: { ...parsed.data, userId: session.user.id }
    })
    return Response.json(created, { status: 201 })
  } catch (e) {
    console.error('Create failed:', e)
    return Response.json({ error: 'Creation failed' }, { status: 500 })
  }
}
```

### Server Actions
```typescript
'use server'

import { auth } from '@/lib/auth'
import { revalidatePath } from 'next/cache'
import { z } from 'zod'

const Schema = z.object({
  title: z.string().min(1).max(200),
})

export async function createItem(formData: FormData) {
  const session = await auth()
  if (!session) return { error: 'Unauthorized' }

  const parsed = Schema.safeParse({
    title: formData.get('title'),
  })

  if (!parsed.success) {
    return { error: 'Validation failed', details: parsed.error.flatten() }
  }

  try {
    await db.item.create({ data: parsed.data })
    revalidatePath('/items')
    return { success: true }
  } catch {
    return { error: 'Failed to create' }
  }
}
```

---

## 4. Database Conventions

### Schema Design
```sql
-- IDs: Use UUIDs or BIGINT (never INT)
id BIGSERIAL PRIMARY KEY

-- Timestamps: Always with timezone
created_at TIMESTAMPTZ DEFAULT NOW()
updated_at TIMESTAMPTZ DEFAULT NOW()

-- Money: Use DECIMAL
price DECIMAL(10,2) NOT NULL

-- Soft delete: Use timestamp
deleted_at TIMESTAMPTZ
```

### Query Patterns
```typescript
// REQUIRED: Include relations (avoid N+1)
const posts = await db.post.findMany({
  include: { author: true, comments: true }
})

// REQUIRED: Paginate lists
const users = await db.user.findMany({
  skip: (page - 1) * limit,
  take: limit,
  orderBy: { createdAt: 'desc' }
})

// REQUIRED: Select only needed fields
const names = await db.user.findMany({
  select: { id: true, name: true }
})
```

---

## 5. Error Handling

### API Errors
```typescript
// Structured error responses
interface ApiError {
  error: string
  code?: string
  details?: unknown
}

// Generic messages to client
return Response.json({ error: 'Operation failed' }, { status: 500 })

// Detailed logs server-side
console.error('Database error:', { userId, operation, error })
```

### UI Errors
```typescript
// Error boundaries for unexpected errors
<ErrorBoundary fallback={<ErrorPage />}>
  <App />
</ErrorBoundary>

// Explicit handling for expected errors
if (error) {
  return (
    <Alert variant="error">
      <AlertTitle>Something went wrong</AlertTitle>
      <AlertDescription>{error.message}</AlertDescription>
      <Button onClick={retry}>Try again</Button>
    </Alert>
  )
}
```

---

## 6. Security Requirements

### Authentication
```typescript
// EVERY protected route
const session = await auth()
if (!session) redirect('/login')
```

### Authorization
```typescript
// EVERY resource access
const resource = await db.resource.findUnique({ where: { id } })
if (resource?.userId !== session.user.id) {
  return notFound() // Don't reveal existence
}
```

### Input Validation
```typescript
// EVERY external input
const schema = z.object({
  email: z.string().email(),
  age: z.number().int().positive().max(150),
})

const result = schema.safeParse(input)
if (!result.success) {
  // Handle validation error
}
```

---

## 7. Testing Standards

### Test Structure
```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('creates user with valid data', async () => {
      const result = await createUser(validData)
      expect(result.success).toBe(true)
    })

    it('rejects invalid email', async () => {
      const result = await createUser({ ...validData, email: 'invalid' })
      expect(result.error).toContain('email')
    })
  })
})
```

### Coverage Requirements
```
Minimum coverage: 80%
Critical paths: 100%
Edge cases: documented and tested
```

---

## 8. Git Workflow

### Commit Messages
```
feat: add user authentication
fix: resolve login redirect loop
refactor: extract validation logic
docs: update API documentation
test: add user service tests
chore: update dependencies
```

### Branch Naming
```
feature/user-authentication
fix/login-redirect
refactor/validation-logic
```

---

## Quick Reference

| Category | Rule |
|----------|------|
| Types | No `any`, explicit signatures |
| Components | All 4 states handled |
| APIs | Auth + Validation + Try/Catch |
| Database | Relations included, paginated |
| Errors | User feedback, server logs |
| Security | Auth on every route |
| Tests | 80% minimum coverage |

---
> Source: [eticmedya/uces](https://github.com/eticmedya/uces) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
