## jitz-journal

> **Next.js (App Router) + PostgreSQL + Prisma + NextAuth.js**

# BJJ Training Tracker — Project Plan

## Recommended Stack

**Next.js (App Router) + PostgreSQL + Prisma + NextAuth.js**

**Rationale:**
- Next.js handles both frontend and backend in a single project, reducing operational complexity.
- Prisma provides type-safe database access and schema migrations for PostgreSQL.
- NextAuth.js handles multi-user authentication with minimal boilerplate and has first-class Next.js support.
- All are production-grade, well-maintained, and have strong community support.

---

## Architecture Overview

```
Browser (React / Next.js)
       |
  Next.js App Router
  ├── /app               → Pages & layouts
  ├── /app/api           → API Route Handlers (REST endpoints)
  ├── /lib               → Prisma client, auth config, utilities
  └── /components        → Shared UI components
       |
  Prisma ORM
       |
  PostgreSQL
```

---

## UI Design Direction

The UI will be built to a high visual standard — not a generic dashboard.

**Key principles:**
- **Design system:** shadcn/ui as the component base, extended with custom styling
- **Styling:** Tailwind CSS with a custom theme (colors, typography, spacing)
- **Charts:** Recharts with custom-styled components matching the theme
- **Motion:** Framer Motion for page transitions and micro-interactions
- **Typography:** Clean hierarchy using a single variable font (Inter or Geist)
- **Layout:** Sidebar navigation on desktop, bottom tab bar on mobile
- **Light/Dark toggle:** System preference as default, user-overridable, persisted to their profile

### Color Palette

| Role | Light | Dark |
|---|---|---|
| Background | `#F8F8F6` | `#0F0F0F` |
| Surface | `#FFFFFF` | `#1A1A1A` |
| Primary | `#C41E3A` (BJJ red) | `#E8294F` |
| Primary foreground | `#FFFFFF` | `#FFFFFF` |
| Muted | `#F1F1EF` | `#242424` |
| Border | `#E4E4E0` | `#2E2E2E` |
| Text primary | `#111111` | `#F2F2F2` |
| Text muted | `#6B6B6B` | `#888888` |

Belt colors render as actual colored badges (white, blue, purple, brown, black) with stripe pip indicators.

### Belt Badge Visual

```
[ BLUE ●●●○○ ]    → Blue belt, 3 stripes
[ PURPLE ●●●●○ ]  → Purple belt, 4 stripes (next = brown)
```

---

## Authentication

### Providers

| Provider | Notes |
|---|---|
| Google OAuth | Standard NextAuth.js provider |
| GitHub OAuth | Standard NextAuth.js provider |
| Email magic link | NextAuth.js Email provider — requires SMTP service (e.g. Resend or SendGrid) |
| Username & password | NextAuth.js Credentials provider + bcrypt hashing |

### Auth Flow

```
/auth/signin
  ├── "Continue with Google"
  ├── "Continue with GitHub"
  ├── "Send magic link" (email input)
  └── "Sign in with email & password"

New user (any provider)
  └── → /onboarding  (profile completion required)

Returning user (onboardingComplete = true)
  └── → /dashboard
```

Password requirements (credentials provider): minimum 12 characters, at least one uppercase, one number, one special character. Enforced with Zod on both client and server.

---

## Onboarding Flow

Until `onboardingComplete = true` is set on the User record, all routes redirect to `/onboarding` via Next.js middleware.

**Step 1 — Create account** (handled by NextAuth.js)

**Step 2 — Complete profile** (required before accessing the app)

### Onboarding Fields

| Field | Type | Required |
|---|---|---|
| displayName | String | Yes |
| dateOfBirth | Date | No |
| heightCm | Decimal | Yes |
| weightKg | Decimal | Yes |
| belt | Enum | Yes |
| stripes | Int (0–4) | Yes |
| gym | String | Yes |
| city | String | No |
| country | String | No |
| trainingStartDate | Date | No |
| preferredTheme | Enum (Light / Dark / System) | Yes |
| avatar | String (URL) | No |

---

## Database Schema

### User

```prisma
model User {
  id                  String    @id @default(uuid())
  email               String    @unique
  passwordHash        String?
  displayName         String
  avatar              String?
  dateOfBirth         DateTime?
  heightCm            Decimal
  weightKg            Decimal
  belt                Belt
  stripes             Int       @default(0) // 0–4
  gym                 String
  city                String?
  country             String?
  trainingStartDate   DateTime?
  preferredTheme      Theme     @default(SYSTEM)
  onboardingComplete  Boolean   @default(false)
  createdAt           DateTime  @default(now())
  updatedAt           DateTime  @updatedAt
}

enum Belt {
  WHITE
  BLUE
  PURPLE
  BROWN
  BLACK
}

enum Theme {
  LIGHT
  DARK
  SYSTEM
}
```

### Belt & Stripe Rules

- Each belt has stripes 0–4 (5 stripes per belt before promotion)
- Stripe 4 = one away from promotion to the next belt
- Black belt is the terminal belt; stripes continue up to degree 6 (coral/red belt can be added later)
- Belt and stripe are displayed everywhere the user appears

### Training Sessions

```prisma
model Session {
  id              String        @id @default(uuid())
  userId          String
  date            DateTime
  durationMinutes Int
  type            SessionType
  gym             String?
  notes           String?
  createdAt       DateTime      @default(now())
  user            User          @relation(fields: [userId], references: [id])
  techniques      Technique[]
  sparringRounds  SparringRound[]
}

enum SessionType {
  GI
  NO_GI
  OPEN_MAT
  DRILLING
  COMPETITION
}
```

### Techniques

```prisma
model Technique {
  id          String          @id @default(uuid())
  userId      String
  sessionId   String?
  name        String
  category    TechniqueCategory
  position    String?
  notes       String?
  learnedAt   DateTime
  user        User            @relation(fields: [userId], references: [id])
  session     Session?        @relation(fields: [sessionId], references: [id])
}

enum TechniqueCategory {
  GUARD
  PASSING
  TAKEDOWNS
  SUBMISSIONS
  ESCAPES
  TRANSITIONS
}
```

### Sparring Rounds

```prisma
model SparringRound {
  id              String        @id @default(uuid())
  sessionId       String
  userId          String
  opponentName    String?
  opponentBelt    Belt?
  result          SparringResult
  submissionType  String?
  notes           String?
  session         Session       @relation(fields: [sessionId], references: [id])
  user            User          @relation(fields: [userId], references: [id])
}

enum SparringResult {
  WIN
  LOSS
  DRAW
}
```

### Physical Metrics

```prisma
model Metric {
  id            String    @id @default(uuid())
  userId        String
  recordedAt    DateTime
  weightKg      Decimal
  bodyFatPct    Decimal?
  notes         String?
  user          User      @relation(fields: [userId], references: [id])
}
```

### Competitions

```prisma
model Competition {
  id          String            @id @default(uuid())
  userId      String
  name        String
  date        DateTime
  location    String?
  weightClass String
  result      CompetitionResult
  notes       String?
  user        User              @relation(fields: [userId], references: [id])
}

enum CompetitionResult {
  GOLD
  SILVER
  BRONZE
  DID_NOT_PLACE
}
```

### Goals & Milestones

```prisma
model Goal {
  id          String      @id @default(uuid())
  userId      String
  title       String
  description String?
  targetDate  DateTime?
  status      GoalStatus  @default(ACTIVE)
  completedAt DateTime?
  user        User        @relation(fields: [userId], references: [id])
}

enum GoalStatus {
  ACTIVE
  COMPLETED
  ABANDONED
}
```

---

## API Routes

All routes are protected. Requests without a valid session return `401`. All write operations enforce row-level ownership — users can only modify their own data.

```
POST   /api/auth/[...nextauth]

GET    /api/sessions
POST   /api/sessions
GET    /api/sessions/[id]
PUT    /api/sessions/[id]
DELETE /api/sessions/[id]

GET    /api/techniques
POST   /api/techniques
PUT    /api/techniques/[id]
DELETE /api/techniques/[id]

GET    /api/sparring
POST   /api/sparring
PUT    /api/sparring/[id]
DELETE /api/sparring/[id]

GET    /api/metrics
POST   /api/metrics
DELETE /api/metrics/[id]

GET    /api/competitions
POST   /api/competitions
PUT    /api/competitions/[id]
DELETE /api/competitions/[id]

GET    /api/goals
POST   /api/goals
PUT    /api/goals/[id]
DELETE /api/goals/[id]
```

---

## Pages / Routes

```
/                        → Dashboard
/sessions                → Session list
/sessions/new            → Log a session
/sessions/[id]           → Session detail (techniques + sparring rounds)
/techniques              → Technique library
/techniques/new          → Log a technique
/sparring                → Sparring history
/metrics                 → Weight & physical metrics
/competitions            → Competition results
/competitions/new        → Log a competition
/goals                   → Goals list
/goals/new               → Create a goal
/onboarding              → Profile completion (new users only)
/auth/signin             → Sign in
/auth/signout            → Sign out
/settings                → Account settings
```

---

## Feature Modules

### Dashboard
- Training frequency over time (bar chart — sessions per week/month)
- Recent sessions list
- Active goals summary
- Current weight trend (line chart)
- Belt and stripe display

### Training Sessions
- Log a session (date, duration, type, gym, notes)
- Link techniques and sparring rounds to a session
- View / edit / delete past sessions

### Techniques
- Log a technique independently or tied to a session
- Browse by category or position
- Search

### Sparring
- Log rounds within a session
- Win / loss / draw tracking per round
- Filter by opponent belt

### Physical Metrics
- Log weight and body fat percentage
- Line chart of weight over time

### Competitions
- Log competition results
- View history sorted by date

### Goals & Milestones
- Create goals with optional target date
- Mark as complete or abandoned
- Belt promotion milestone tracking

---

## UI Component Inventory

### Global
- `Sidebar` — desktop nav with user avatar, belt badge, active route highlight
- `BottomTabBar` — mobile nav
- `ThemeToggle` — sun/moon icon, persists preference to user profile
- `BeltBadge` — colored belt with stripe pip indicators
- `Avatar` — with fallback initials
- `PageHeader` — title + optional action button
- `StatCard` — metric display (sessions this month, total hours, etc.)
- `EmptyState` — illustrated placeholder when no data exists
- `LoadingSkeleton` — per-component loading states

### Forms
- `SessionForm`
- `TechniqueForm`
- `SparringRoundForm`
- `MetricForm`
- `CompetitionForm`
- `GoalForm`
- `ProfileForm` (used in onboarding and settings)

### Data Display
- `SessionCard`
- `TechniqueCard`
- `SparringResultBadge` — Win / Loss / Draw pill
- `GoalProgressBar`
- `WeightChart` — Recharts line chart
- `TrainingFrequencyChart` — Recharts bar chart
- `BeltProgressTracker` — visual belt history timeline

---

## Security Considerations

- All API routes validate the session and enforce that the requesting user owns the resource (row-level ownership checks via Prisma `where: { id, userId }`)
- Passwords are hashed with bcrypt (minimum cost factor 12) for the credentials provider
- OAuth providers (Google, GitHub) never store passwords
- Environment variables (DB connection string, NextAuth secret, SMTP credentials, OAuth client secrets) are never committed to source control — use `.env.local`
- Prisma parameterized queries prevent SQL injection by default
- CSRF protection is built into NextAuth.js
- Zod validation runs on both client and server for all form inputs

---

## Recommended Third-Party Libraries

| Purpose | Library |
|---|---|
| ORM | Prisma |
| Auth | NextAuth.js |
| Charts | Recharts |
| Form state | React Hook Form |
| Validation | Zod |
| UI components | shadcn/ui |
| Animation | Framer Motion |
| Date handling | date-fns |
| Email (magic link) | Resend or SendGrid |
| Styling | Tailwind CSS |

---

## Build Order

1. Project scaffolding — Next.js App Router, Tailwind, shadcn/ui, Framer Motion, Prisma, PostgreSQL
2. Custom Tailwind theme — colors, typography, dark/light CSS variables
3. NextAuth.js setup — Google, GitHub, email magic link, and credentials providers
4. Database schema + migrations (full User profile, all feature tables)
5. Onboarding flow — profile form, middleware redirect guard
6. Core layout — Sidebar, BottomTabBar, ThemeToggle, BeltBadge components
7. Dashboard — stat cards, training frequency chart, weight trend
8. Training sessions CRUD
9. Techniques — log and browse
10. Sparring — log rounds, win/loss/draw tracking
11. Physical metrics — log entries, weight chart
12. Competitions — log and view history
13. Goals — create, complete, abandon
14. Polish — page transitions, empty states, loading skeletons, error boundaries

---
> Source: [cooperbraun13/jitz-journal](https://github.com/cooperbraun13/jitz-journal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
