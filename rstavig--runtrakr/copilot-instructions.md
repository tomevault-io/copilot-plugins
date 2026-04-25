## runtrakr

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RunTrakr is a Next.js 15 fitness tracking application for logging workout runs (Hills, Dots), general workouts, and races. Built with TypeScript, Tailwind CSS, shadcn/ui components, Prisma ORM with PostgreSQL (Neon), and NextAuth v5 for authentication.

## Development Commands

### Essential Commands
```bash
# Development server with Turbopack
npm run dev

# Type checking (critical before commits)
npx tsc --noEmit

# Build for production
npm run build

# Linting
npm run lint

# Database operations
npx prisma generate          # Generate Prisma client
npx prisma db push          # Push schema changes to database
npx prisma studio           # Open Prisma Studio GUI
ts-node db/seed.ts          # Seed database with sample data
```

## Architecture

### Authentication Flow
- **NextAuth v5** (`auth.ts`) with Credentials provider
- JWT session strategy (30-day expiration)
- Password hashing: `bcrypt-ts-edge`
- Session/token callbacks extend user object with `id` and `name`
- Protected routes via layouts (see route groups below)

### Route Structure
The app uses Next.js 15 App Router with route groups:

- **`app/(auth)/`** - Public authentication pages (signin, signup)
- **`app/dashboard/`** - User dashboard for tracking workouts
  - `/hills` - Hill runs tracking
  - `/dots` - Dot runs tracking
  - `/workouts` - General workouts
  - `/runs`, `/races` - Additional tracking
  - `/test` - Test CRUD operations (development/learning)
- **`app/admin/`** - Admin-only user management

Each route group has its own `layout.tsx` controlling auth/access.

### Data Layer Pattern

All database operations follow this server action pattern:

1. **Validators** (`lib/validators.ts`) - Zod schemas for all forms
2. **Actions** (`lib/actions/*.ts`) - Server actions marked with `'use server'`
   - `userActions.ts` - User CRUD
   - `hillActions.ts` - Hill runs CRUD
   - `dotActions.ts` - Dot runs CRUD
   - `workoutActions.ts` - Workouts CRUD
   - `testActions.ts` - Test items CRUD
3. **Pattern**: All actions accept `(prevState: unknown, formData: FormData)` signature
4. **Return type**: `{ success: boolean; message: string }` for consistency
5. **Post-mutation**: Call `revalidatePath()` and `redirect()` after successful writes

### Form Handling

Forms use `react-hook-form` + `zodResolver` + `useActionState`:

```typescript
// Component pattern
const form = useForm<z.infer<typeof schema>>({
  resolver: zodResolver(schema),
  defaultValues: { ... }
})

const [state, action, isPending] = useActionState(serverAction, {
  success: false,
  message: ''
})

// Form submission
<Form {...form}>
  <form action={action}>
    {/* FormField components */}
  </form>
</Form>
```

### Database Schema (Prisma)

**Key Models**:
- **User** - UUID primary key, email (unique), hashed password, role field
- **Session** - NextAuth session storage
- **Hills** - Integer ID (autoincrement), date (@db.Date), et/best as Float
- **Dots** - UUID ID, daterun, et/best as String, loops as Int
- **Workouts** - UUID ID, workoutDate, various exercise counts
- **Test** - UUID ID, sample CRUD model for learning

**Important**:
- Prisma configured with `driverAdapters` preview feature for Neon serverless
- Client initialized in `db/prisma.ts` with `withAccelerate()` extension
- Use `convertToPlainObject()` from `lib/utils.ts` when returning Prisma results from server actions

### Type Handling Gotchas

1. **Date fields**: Schemas define dates as `z.coerce.date()` but forms may need Date objects (not ISO strings) as defaultValues
2. **Hills model**: Uses `date: DateTime @db.Date()` which returns Date objects - convert to strings in page components when needed
3. **Server action returns**: Must match `{ success: boolean; message: any }` for `useActionState` compatibility
4. **Import types**: DeleteDialog and similar components export as default, not named exports

### Utility Functions (`lib/utils.ts`)

- **`convertToPlainObject<T>(value: T)`** - Serialize Prisma objects (removes proxies/metadata)
- **`formatError(error)`** - Unified error formatting for Zod/Prisma/generic errors
- **`formatId(id: string)`** - Shorten UUIDs for display (`..abc123`)
- **`formatDateTime(dateString: Date)`** - Returns `{ dateTime, dateOnly, timeOnly }`

### UI Components

- **shadcn/ui** components in `components/ui/`
- Custom components in `app/dashboard/components/`
- Form pattern: Use FormField with Controller from react-hook-form
- Date pickers: Custom DatePicker component wrapping react-day-picker

## Common Workflows

### Adding a New Tracking Feature

1. Create Prisma model in `prisma/schema.prisma`
2. Run `npx prisma db push` and `npx prisma generate`
3. Create Zod schema in `lib/validators.ts`
4. Create server actions in `lib/actions/[feature]Actions.ts`
5. Create page routes under `app/dashboard/[feature]/`
6. Create form component using react-hook-form + useActionState pattern
7. Test with `npx tsc --noEmit` before committing

### Seeding Database

Edit `db/sample-data.ts` then run `ts-node db/seed.ts`. Uses `hashSync()` from bcrypt-ts-edge for password hashing (synchronous function, no await needed).

## Environment Variables

Required in `.env`:
- `DATABASE_URL` - Neon PostgreSQL connection string
- `AUTH_SECRET` - NextAuth secret key
- NextAuth auto-configuration via `auth.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rstavig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
