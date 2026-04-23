## university-applications-organizer

> A full-stack web app for tracking 20+ university applications. Students can manage universities, track requirements (essays, test scores, recommendations), manage deadlines, and view a calendar timeline. Data is fully isolated per user via Clerk authentication.

# University Applications Organizer — Claude Context

## What This Project Is
A full-stack web app for tracking 20+ university applications. Students can manage universities, track requirements (essays, test scores, recommendations), manage deadlines, and view a calendar timeline. Data is fully isolated per user via Clerk authentication.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 15 (App Router) |
| Language | TypeScript 5.7 |
| Database ORM | Prisma 6 |
| Database | Neon PostgreSQL (production) |
| Auth | Clerk (`@clerk/nextjs` v7) |
| Styling | Tailwind CSS 3.4 |
| UI Primitives | Radix UI (checkbox, dialog, dropdown, label, select, slot) |
| Icons | Lucide React |
| Date Picker | react-day-picker |
| Utilities | clsx, tailwind-merge, class-variance-authority |
| Runtime | Node.js 18.17+ |

---

## Features Implemented

### Authentication (Clerk)
- Sign in / Sign up at `/sign-in` and `/sign-up` (embedded Clerk components)
- OAuth providers: Google, Apple, Microsoft (Outlook/Azure AD), Email/Password
- Middleware protects all routes except `/sign-in` and `/sign-up`
- Custom nav user menu: orange circle with initials, dropdown shows name/email + sign-out
- Per-user data isolation: every Prisma query is scoped by `userId` from `auth()`

### Dashboard (`/`)
- Stats overview: total universities, count by status (considering/applied/accepted/waitlisted/rejected/enrolled)
- Upcoming deadlines widget with 7/14/30-day filter tabs
- Quick actions: Add University, View All Universities, Timeline
- Status cards are clickable (links to universities page filtered by that status)

### Universities (`/universities`)
- Card grid with color-coded top stripe by category (reach=red, target=blue, safety=green)
- Status and category badges on each card
- Next deadline + requirements progress bar per card
- Search by name, filter by status and category
- Add/Edit modal form (full-screen, all fields)
- University search: US College Scorecard API + Hipolabs worldwide API (auto-fills name, location, website)

### University Detail (`/universities/[id]`)
- Edit status and category inline
- Requirements checklist: add/edit/delete, toggle completion, progress bar
- Deadlines manager: add/edit/delete, toggle completion, color-coded urgency (red/orange/blue/green)
- Notes section: view/edit toggle, character count

### Timeline (`/timeline`)
- Monthly calendar view of all deadlines
- Color-coded urgency: red (urgent ≤3 days), orange (4–7 days), blue (normal)
- Filter by university or deadline type
- Countdown (days until each deadline)

### University Comparison
- Side-by-side comparison of multiple universities
- Key metrics: rankings, acceptance rates, costs, deadlines

---

## Database Schema

### University
```prisma
model University {
  id                    String      @id @default(cuid())
  userId                String                          // Clerk user ID — all queries scoped by this
  name                  String
  location              String
  program               String
  status                String      @default("considering")
  // "considering" | "applied" | "accepted" | "rejected" | "waitlisted" | "enrolled"
  category              String?     // "reach" | "target" | "safety"
  ranking               Int?
  acceptanceRate        Float?
  websiteUrl            String?
  notes                 String?
  researchNotes         String?
  applicationDeadline   DateTime?
  earlyDeadline         DateTime?
  decisionDate          DateTime?
  tuition               Float?
  applicationFee        Float?
  estimatedCostOfLiving Float?
  financialAidDeadline  DateTime?
  scholarshipNotes      String?
  createdAt             DateTime    @default(now())
  updatedAt             DateTime    @updatedAt
  requirements          Requirement[]
  deadlines             Deadline[]
  @@index([userId])
}
```

### Requirement
```prisma
model Requirement {
  id           String    @id @default(cuid())
  universityId String
  type         String    // essay, test_score, recommendation, transcript, portfolio, other
  title        String
  description  String?
  completed    Boolean   @default(false)
  deadline     DateTime?
  notes        String?
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt
  university   University @relation(fields: [universityId], references: [id], onDelete: Cascade)
  @@index([universityId])
}
```

### Deadline
```prisma
model Deadline {
  id           String    @id @default(cuid())
  universityId String
  title        String
  date         DateTime
  type         String    // application, financial_aid, scholarship, decision, deposit, housing, other
  description  String?
  completed    Boolean   @default(false)
  reminderDays Int?
  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt
  university   University @relation(fields: [universityId], references: [id], onDelete: Cascade)
  @@index([universityId])
  @@index([date])
}
```

---

## Environment Variables

Create a `.env` file at the project root (never commit real secrets):

```env
# Neon PostgreSQL
DATABASE_URL="postgresql://user:password@host.neon.tech/dbname?sslmode=require"

# College Scorecard API (optional — DEMO_KEY works with lower rate limits)
COLLEGE_SCORECARD_API_KEY=DEMO_KEY

# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_...
CLERK_SECRET_KEY=sk_test_...
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/
```

All 6 Clerk variables plus `DATABASE_URL` must also be set in Vercel Environment Variables before deploying.

---

## How to Run Locally

```bash
# 1. Install dependencies
npm install

# 2. Create .env and fill in DATABASE_URL + Clerk keys (see above)

# 3. Push schema to your database (first time)
npx prisma db push

# 4. (Optional) Open Prisma Studio to inspect data
npm run db:studio

# 5. Start dev server
npm run dev
```

Open http://localhost:3000 — Clerk will redirect to `/sign-in` on first visit.

### Useful Scripts
| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server |
| `npm run build` | Build for production |
| `npm run lint` | Run ESLint |
| `npm run db:migrate` | Run Prisma migrations |
| `npm run db:seed` | Seed sample data (5 universities) |
| `npm run db:reset` | Reset database (deletes all data!) |
| `npm run db:studio` | Open Prisma Studio GUI |

---

## Deployment — Vercel

The app is deployed on **Vercel** connected to this GitHub repository.

1. Push to `main` → Vercel auto-deploys
2. Required environment variables in Vercel project settings:
   - `DATABASE_URL` (Neon connection string)
   - `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
   - `CLERK_SECRET_KEY`
   - `NEXT_PUBLIC_CLERK_SIGN_IN_URL` → `/sign-in`
   - `NEXT_PUBLIC_CLERK_SIGN_UP_URL` → `/sign-up`
   - `NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL` → `/`
   - `NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL` → `/`
   - `COLLEGE_SCORECARD_API_KEY` (optional, `DEMO_KEY` works)

---

## Database — Neon PostgreSQL

- Provider: [Neon](https://console.neon.tech) (serverless PostgreSQL)
- Schema is pushed via `npx prisma db push` (no migration files needed for Neon)
- The `University` table has a `userId` column (added when Clerk was integrated)
- Existing seed data was wiped before the `userId` column was added (required for non-nullable migration)

---

## Auth — Clerk Setup

1. Create app at [clerk.com](https://clerk.com), name it "University Applications Organizer"
2. Enable sign-in methods: Email, Google, Apple, Microsoft
3. For Apple: Clerk Dashboard → User & Authentication → Social Connections → Apple (requires Apple Developer account)
4. For Microsoft: Clerk Dashboard → Social Connections → Microsoft (requires Azure AD app registration)
5. Copy Publishable Key (`pk_test_...`) and Secret Key (`sk_test_...`) into `.env` and Vercel

### How Auth Works in Code
- `middleware.ts` (project root): protects all routes, allows `/sign-in` and `/sign-up` through
- `app/layout.tsx`: wraps app in `<ClerkProvider>`
- `app/sign-in/[[...sign-in]]/page.tsx` + `app/sign-up/[[...sign-up]]/page.tsx`: embedded Clerk UI
- API routes: call `auth()` from `@clerk/nextjs/server`, return 401 if no `userId`, scope all Prisma queries by `userId`
- `components/layout/Navigation.tsx`: uses `useAuth()` (`isLoaded` guard), `useUser()`, `useClerk()`, `SignedIn`/`SignedOut` components; hamburger menu on mobile (≤md) opens full-width overlay with nav links + user info + sign out

---

## Project Structure

```
university-applications-organizer/
├── app/
│   ├── api/
│   │   ├── universities/route.ts          GET all, POST new (userId scoped)
│   │   ├── universities/[id]/route.ts     GET, PATCH, DELETE by ID (userId scoped)
│   │   ├── requirements/route.ts          POST new
│   │   ├── requirements/[id]/route.ts     PATCH, DELETE
│   │   ├── deadlines/route.ts             POST new
│   │   ├── deadlines/[id]/route.ts        PATCH, DELETE
│   │   ├── stats/route.ts                 GET dashboard stats (userId scoped)
│   │   └── search-universities/route.ts   GET — searches College Scorecard + Hipolabs
│   ├── sign-in/[[...sign-in]]/page.tsx
│   ├── sign-up/[[...sign-up]]/page.tsx
│   ├── universities/page.tsx              List page (client component)
│   ├── universities/[id]/page.tsx         Detail page (client component)
│   ├── timeline/page.tsx
│   ├── layout.tsx                         ClerkProvider wrapper
│   └── page.tsx                           Dashboard (server component)
├── components/
│   ├── layout/Navigation.tsx              Nav with Clerk user menu
│   ├── layout/PageHeader.tsx
│   ├── universities/UniversityCard.tsx
│   ├── universities/UniversityFilters.tsx
│   ├── universities/UniversityForm.tsx
│   ├── detail/UniversityHeader.tsx
│   ├── detail/RequirementsChecklist.tsx
│   ├── detail/DeadlinesManager.tsx
│   ├── detail/NotesSection.tsx
│   └── dashboard/ (StatsOverview, UpcomingDeadlines, QuickActions)
├── prisma/
│   ├── schema.prisma
│   └── seed.ts
├── lib/db.ts                              Prisma client singleton
├── types/index.ts                         TypeScript types
├── middleware.ts                          Clerk route protection
├── plans/                                 Feature plan documents
└── CLAUDE.md                              This file
```

---

## Current Project Status (as of 2026-03-29)

- [x] Initial app: Next.js 15 + Prisma + SQLite + all UI components
- [x] All pages connected to API routes (dashboard, universities list, detail, timeline)
- [x] Deployed to Vercel
- [x] Migrated database from SQLite to Neon PostgreSQL
- [x] Added Clerk authentication with per-user data isolation
- [x] Full mobile responsiveness (hamburger nav, responsive layouts, touch-friendly UI)
- [ ] Email/calendar reminders for deadlines
- [ ] PDF export of application summary
- [ ] Drag-and-drop requirement ordering

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coderAshraful) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
