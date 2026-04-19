## dentalize

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dentalize is a task management web application tailored for small dental practices. Built with Next.js 15, it features user authentication, a 7-day calendar interface, and management for Services, Clients, and Tasks. The entire UI is in Brazilian Portuguese (pt-BR).

## Technology Stack

- **Framework**: Next.js 15 with App Router
- **Language**: TypeScript
- **Database**: SQLite with Prisma ORM (easily migrated to PostgreSQL)
- **Authentication**: NextAuth.js v5 with credentials provider
- **UI**: shadcn/ui components + Tailwind CSS
- **Date handling**: date-fns with pt-BR locale
- **Validation**: Zod schemas

## Development Commands

```bash
# Install dependencies
npm install

# Run development server (http://localhost:3000)
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Database operations
export $(cat .env.local | xargs) && npx prisma studio  # Open Prisma Studio
export $(cat .env.local | xargs) && npx prisma migrate dev  # Create migration
export $(cat .env.local | xargs) && npx prisma generate  # Generate Prisma client
```

Note: SQLite requires environment variables to be loaded. Prefix Prisma commands with `export $(cat .env.local | xargs) &&`.

## Architecture Overview

### Data Model

Three core entities with relationships:

- **User**: Authentication and task ownership
- **Client**: Dental practice patients (optional relationship to tasks)
- **Service**: Procedures with duration, price, and color coding (optional relationship to tasks)
- **Task**: Scheduled appointments/events with startTime, endTime, status, and optional client/service

Task statuses: SCHEDULED, IN_PROGRESS, COMPLETED, CANCELLED

### Directory Structure

```
app/
├── (auth)/          # Grouped route for unauthenticated pages
│   ├── login/
│   └── register/
├── dashboard/       # Protected routes
│   ├── layout.tsx   # Sidebar navigation
│   ├── page.tsx     # Main 7-day calendar view
│   ├── clientes/    # Client management
│   └── servicos/    # Service management
└── api/auth/        # NextAuth API routes

components/
├── ui/              # shadcn/ui base components
├── calendar/        # WeekView and TaskCard components
├── tasks/           # TaskForm modal
├── clients/         # (Future: client-specific components)
└── services/        # (Future: service-specific components)

actions/             # Server Actions for data mutations
├── auth.ts          # register, login, logout
├── tasks.ts         # CRUD for tasks
├── clients.ts       # CRUD for clients
└── services.ts      # CRUD for services

lib/
├── auth.ts          # NextAuth configuration
├── prisma.ts        # Prisma client singleton
└── utils.ts         # Utility functions (cn for className merging)

middleware.ts        # Route protection and authentication checks
```

### Key Architectural Patterns

1. **Server Actions**: All data mutations use Next.js Server Actions instead of API routes. This provides type-safe mutations with automatic revalidation.

2. **Protected Routes**: Middleware checks authentication for `/dashboard/*`, `/clientes/*`, `/servicos/*`, and `/tarefas/*` routes. Redirects to `/login` if unauthenticated.

3. **Route Groups**: `(auth)` and `dashboard` group routes with shared layouts without affecting URL structure.

4. **Prisma Singleton**: `lib/prisma.ts` ensures a single Prisma client instance in development to avoid connection exhaustion.

5. **Optimistic UI**: Dashboard pages fetch data client-side for interactivity, then revalidate after mutations using `revalidatePath()`.

## Calendar Implementation

### 7-Day Week View Logic

- **Week Start**: Sunday (Brazilian standard) calculated via `startOfWeek(date, { weekStartsOn: 0 })`
- **Business Hours**: 7:00 AM to 7:00 PM (12 hours)
- **Time Slots**: 30-minute intervals (24 slots per day)
- **Task Positioning**: Tasks are absolutely positioned within their time slot based on `startTime`, with height calculated from duration (`endTime - startTime`)
- **Color Coding**: Tasks inherit color from associated Service, or default to blue
- **Interactions**:
  - Click empty slot → Opens TaskForm with pre-filled date/time
  - Click task → Opens TaskForm in edit mode
  - Week navigation → Refetches tasks for new date range

### Week Calculation Flow

1. Get current date
2. Calculate `startOfWeek` (Sunday)
3. Generate array of 7 days: `[Sunday, Monday, ..., Saturday]`
4. Fetch tasks where `startTime >= weekStart && startTime <= weekEnd`
5. Render grid: 7 columns (days) × 24 rows (30-min slots)

## Authentication Flow

1. **Register**: User submits form → `registerUser` action hashes password with bcrypt → Creates user in DB → Redirects to login
2. **Login**: User submits credentials → NextAuth validates via Credentials provider → Compares hashed password → Creates JWT session → Redirects to `/dashboard`
3. **Logout**: User clicks logout → `logoutUser` action → Session cleared → Redirects to `/login`
4. **Session Management**: Middleware checks `req.auth` on protected routes

## Common Tasks

### Adding a New Server Action

1. Create action in `actions/` directory
2. Add `"use server"` directive at top of file
3. Use Zod for validation
4. Check authentication with `await auth()`
5. Call `revalidatePath()` after mutations

Example:
```typescript
"use server"
import { auth } from "@/lib/auth"
import { prisma } from "@/lib/prisma"
import { revalidatePath } from "next/cache"

export async function myAction(formData: FormData) {
  const session = await auth()
  if (!session?.user?.id) {
    return { error: "Não autenticado" }
  }

  // Perform database operation
  await prisma.model.create({ data: { ... } })

  revalidatePath("/your-page")
  return { success: true }
}
```

### Migrating to PostgreSQL

1. Update `.env.local`:
   ```
   DATABASE_URL="postgresql://user:password@localhost:5432/dentalize"
   ```

2. Update `prisma/schema.prisma`:
   ```prisma
   datasource db {
     provider = "postgresql"
     url      = env("DATABASE_URL")
   }
   ```

3. Run migration:
   ```bash
   export $(cat .env.local | xargs) && npx prisma migrate dev
   ```

### Adding a New UI Component

1. Components follow shadcn/ui patterns
2. Use `cn()` utility for className merging
3. All text must be in Portuguese
4. Use Tailwind CSS for styling
5. Add to `components/ui/` for reusable components

## Important Conventions

- **Language**: All UI text, labels, error messages, and comments in Brazilian Portuguese
- **Date Format**: Use date-fns with `{ locale: ptBR }`
- **Status Labels**: Map enum values to Portuguese in UI layer
- **Form Validation**: Use Zod schemas in server actions, display Portuguese error messages
- **Soft Deletes**: Client/Service deletion sets related tasks to `null` (not cascade delete)
- **Time Format**: 24-hour format (e.g., "14:30")
- **Date Format**: "15 de janeiro de 2026" for full dates, "Dom, 15 Jan" for calendar headers

## Database Schema Notes

- Use `@default(cuid())` for IDs (collision-resistant)
- Task `startTime` and `endTime` are DateTime (stored as ISO strings)
- Service `duration` is in minutes (Integer)
- Service `price` is Float (SQLite doesn't have Decimal type, use Float)
- Indexes on `[userId, startTime]` and `[startTime, endTime]` for query performance

## Troubleshooting

- **Prisma errors**: Ensure environment variables are loaded with `export $(cat .env.local | xargs)`
- **Database locked**: Only one Prisma Studio instance can run at a time
- **NextAuth session issues**: Check `NEXTAUTH_SECRET` is set in `.env.local`
- **Build warnings about Edge Runtime**: Safe to ignore, middleware uses Node.js runtime

## Future Enhancements (Not Yet Implemented)

- Drag-and-drop task rescheduling
- Recurring appointments
- Email/SMS notifications
- Multi-user support with role-based access
- Financial reporting
- Client appointment history view

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Possomato) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
