## makerspace-rsvp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Context

This is a **multi-event RSVP, waiver signing, and attendance tracking system** for **Fubar Labs** (New Jersey's first hackerspace) built with **Remix** and deployed to **Cloudflare Pages**. The system manages robot combat events, maker faires, workshops, and community gatherings with persistent database storage.

When working with this codebase:
- **Prioritize test-driven development (TDD)** - Write tests before implementation
- Use Git and GitHub CLI (`gh`) for all version control operations
- Ask clarifying questions before making architectural changes
- Read relevant files thoroughly before proposing changes

## Key Features

### User-Facing
- Browse upcoming events (robot combat, workshops, maker faires)
- RSVP with email confirmation
- Digital waiver signing with consent tracking
- QR code generation for event check-ins
- View personal RSVP history
- Email notifications and reminders

### Admin-Facing (Protected by Login)
- Create and manage events
- View RSVP lists and statistics
- Check attendees in (manual or QR code scan)
- Export attendance data
- Send bulk emails to attendees
- Manage waitlists and capacity

### Future Enhancements
- Discord integration for announcements
- Claude AI agent for answering event questions
- Real-time check-in dashboard (Durable Objects)
- Stripe integration for paid events
- Equipment reservation system

## Architecture Overview

### Technology Stack
- **Framework**: Remix 2.x (React Router v6)
- **UI**: React 18 + TypeScript + Tailwind CSS + shadcn/ui
- **Database**: Cloudflare D1 (SQLite) or Turso
- **ORM**: Drizzle ORM (type-safe, lightweight)
- **Deployment**: Cloudflare Pages + Workers
- **Auth**: Session-based (Cloudflare KV for sessions)
- **Email**: Resend or SendGrid
- **Package Manager**: pnpm (fast, efficient)

### Request Flow
```
User Browser
  ↓
Remix App (Cloudflare Pages)
  ↓
Loaders (Data Fetching)
  ↓
Database (D1 or Turso)
  ↓
Actions (Form Submissions)
  ↓
Server-Side Logic
  ↓
Response (SSR or JSON)
```

### Directory Structure
```
app/
├── routes/                    # File-based routing
│   ├── _index.tsx            # Homepage - upcoming events list
│   ├── events.$id.tsx        # Event detail + RSVP form
│   ├── events.$id.waiver.tsx # Waiver signing page
│   ├── events.$id.checkin.tsx # QR code check-in page
│   ├── my-rsvps.tsx          # User's RSVP history
│   ├── admin.tsx             # Admin layout with auth check
│   ├── admin._index.tsx      # Admin dashboard
│   ├── admin.events.tsx      # Event management
│   ├── admin.events.new.tsx  # Create new event
│   ├── admin.events.$id.tsx  # Edit event + attendee list
│   └── admin.login.tsx       # Admin login
├── components/
│   ├── ui/                   # shadcn/ui components
│   ├── EventCard.tsx         # Event preview card
│   ├── RSVPForm.tsx          # Reusable RSVP form
│   ├── WaiverForm.tsx        # Digital waiver component
│   ├── QRCodeDisplay.tsx     # QR code generator
│   ├── CheckInScanner.tsx    # QR scanner for check-ins
│   └── AdminNav.tsx          # Admin navigation
├── lib/
│   ├── db.server.ts          # Database connection + Drizzle setup
│   ├── schema.ts             # Drizzle schema definitions
│   ├── auth.server.ts        # Session management
│   ├── email.server.ts       # Email sending utilities
│   └── qr.server.ts          # QR code generation
├── utils/
│   ├── validation.ts         # Zod schemas for forms
│   └── date.ts               # Date formatting helpers
└── root.tsx                  # App shell, meta tags, global styles

db/
├── migrations/               # SQL migration files
└── seed.ts                  # Sample data for development

tests/
├── routes/                  # Route tests (loaders, actions)
├── components/              # Component tests
└── e2e/                     # Playwright end-to-end tests

public/
├── favicon.ico
└── robots.txt
```

### Key Architectural Patterns
- **File-Based Routing**: Routes map directly to URLs
- **Loader Pattern**: Server-side data fetching before render
- **Action Pattern**: Server-side form handling with progressive enhancement
- **Nested Layouts**: Shared UI components (admin layout, event layout)
- **Type Safety**: End-to-end TypeScript with Drizzle ORM
- **Progressive Enhancement**: Forms work without JavaScript

## Common Commands

```bash
# Setup
pnpm install                  # Install dependencies

# Development
pnpm dev                      # Start dev server (localhost:3000)
pnpm typecheck                # Run TypeScript type checking

# Database
pnpm db:generate              # Generate migrations from schema
pnpm db:migrate               # Run migrations (local)
pnpm db:migrate:prod          # Run migrations (production D1)
pnpm db:seed                  # Seed database with sample data
pnpm db:studio                # Open Drizzle Studio (database GUI)

# Testing (ALWAYS run before committing)
pnpm test                     # Run all tests (Vitest)
pnpm test:watch               # Run tests in watch mode
pnpm test:e2e                 # Run Playwright E2E tests
pnpm test:e2e:ui              # Run E2E tests with UI
pnpm test:coverage            # Run tests with coverage report

# Code Quality
pnpm lint                     # ESLint + Prettier check
pnpm format                   # Format code with Prettier
pnpm validate                 # Run typecheck + lint + test

# Build & Deploy
pnpm build                    # Build for production
pnpm preview                  # Preview production build locally
pnpm deploy                   # Deploy to Cloudflare Pages
pnpm cf:tail                  # Tail Cloudflare logs

# Git and GitHub (using gh CLI)
gh pr create --title "feat: add waiver signing" --body "Description"
gh pr list
gh pr status
gh issue create --title "Bug: RSVP form validation" --body "Steps to reproduce"
```

## Development Workflows

### Standard Feature Development (Explore-Plan-Code-Commit)

ALWAYS follow this workflow for new features:

1. **Explore**: Read relevant files without writing code yet
   - Use subagents to investigate specific questions
   - Understand existing architecture and patterns
   - Check similar implementations in the codebase
   
2. **Plan**: Create detailed implementation plan
   - Use "think" or "think hard" for complex problems
   - Document plan in markdown file or GitHub issue
   - Consider database changes, routes, components needed
   - Get confirmation before proceeding
   
3. **Code**: Implement the solution
   - Write tests first (TDD)
   - Implement loaders and actions
   - Create UI components
   - Run tests frequently
   
4. **Commit**: Create commit and PR
   - Use conventional commit format (feat:, fix:, test:, etc.)
   - Update documentation if needed
   - Ensure all tests pass

### Test-Driven Development (PREFERRED for this project)

**CRITICAL**: This project follows TDD. ALWAYS write tests before implementation.

1. **Write Tests First**
   ```bash
   # Tell Claude explicitly:
   # "Write tests for X using TDD. Do not create mock implementations."
   ```
   - Write loader tests (data fetching)
   - Write action tests (form submissions)
   - Write component tests (UI behavior)
   - Ensure tests fail initially (Red phase)
   - Commit tests separately
   
2. **Implement Code**
   ```bash
   # Tell Claude explicitly:
   # "Now implement code to pass the tests. Do not modify the tests."
   ```
   - Write minimal code to pass tests (Green phase)
   - Implement loaders, actions, components
   - Iterate until all tests pass
   
3. **Refactor**
   - Improve code quality while keeping tests green
   - Extract reusable components
   - Run format command
   - Commit implementation

### Git Workflow

**Branch Naming**:
- `feature/*` - New features (e.g., `feature/waiver-signing`)
- `fix/*` - Bug fixes (e.g., `fix/rsvp-validation`)
- `test/*` - Test improvements
- `refactor/*` - Code refactoring
- `docs/*` - Documentation updates

**Commit Message Format** (Conventional Commits):
```
feat: add digital waiver signing flow
fix: resolve RSVP capacity validation error
test: add E2E tests for check-in workflow
refactor: extract QR code component
docs: update CLAUDE.md with deployment steps
chore: update dependencies
```

**Pre-Commit Checklist**:
```bash
pnpm validate              # Runs typecheck + lint + test
git status                 # Review staged changes
git diff                   # Review actual changes
```

## Code Standards

### General Guidelines
- **TypeScript strict mode** - All files must type-check
- **ESLint + Prettier** - Enforced via pre-commit hooks
- **Functional components** - Use React hooks, not class components
- **Server-side first** - Use loaders/actions before client-side fetching
- **Progressive enhancement** - Forms should work without JavaScript
- **Accessibility** - WCAG 2.1 AA compliance (keyboard nav, ARIA labels)

### Testing Requirements
- **>80% code coverage** overall
- **100% coverage** for critical paths (RSVP, waiver, check-in)
- **All loaders and actions** must have tests
- **All form validations** must have tests
- **Mock external services** (email, QR generation) in tests
- **E2E tests** for complete user flows

### Component Development Pattern
When adding new components:
1. Write test first (TDD)
2. Use TypeScript for props
3. Add JSDoc comments for complex components
4. Use shadcn/ui primitives when possible
5. Ensure keyboard accessibility
6. Add loading and error states
7. Test on mobile viewport

### Route Development Pattern
When adding new routes:
1. Define loader for data fetching (if needed)
2. Define action for form handling (if needed)
3. Validate inputs with Zod schemas
4. Handle errors gracefully
5. Return proper HTTP status codes
6. Add meta tags for SEO
7. Test loader and action separately

### Database Patterns
- Use Drizzle ORM for all queries (no raw SQL unless necessary)
- Use transactions for multi-step operations
- Index frequently queried columns
- Use foreign keys for relationships
- Validate constraints at database level
- Use prepared statements to prevent SQL injection
- Test all CRUD operations in isolation

## Database Schema

### Events
```typescript
events {
  id: string (uuid)
  name: string
  slug: string (unique, url-friendly)
  description: text
  date: date
  timeStart: time
  timeEnd: time
  location: string
  capacity: integer (nullable)
  requiresWaiver: boolean
  waiverText: text (nullable)
  discordLink: string (nullable)
  status: enum ['draft', 'published', 'cancelled']
  createdAt: timestamp
  updatedAt: timestamp
}
```

### Attendees
```typescript
attendees {
  id: string (uuid)
  email: string (unique)
  name: string
  createdAt: timestamp
}
```

### RSVPs
```typescript
rsvps {
  id: string (uuid)
  eventId: string (FK → events.id)
  attendeeId: string (FK → attendees.id)
  status: enum ['yes', 'no', 'maybe', 'waitlist', 'cancelled']
  notes: text (nullable)
  createdAt: timestamp
  updatedAt: timestamp
  
  UNIQUE (eventId, attendeeId)
}
```

### Waivers
```typescript
waivers {
  id: string (uuid)
  eventId: string (FK → events.id)
  attendeeId: string (FK → attendees.id)
  waiverText: text (snapshot of waiver at signing time)
  signedAt: timestamp
  ipAddress: string
  consent: boolean (must be true)
  
  UNIQUE (eventId, attendeeId)
}
```

### Attendance
```typescript
attendance {
  id: string (uuid)
  eventId: string (FK → events.id)
  attendeeId: string (FK → attendees.id)
  checkedInAt: timestamp
  checkInMethod: enum ['manual', 'qr_code', 'email_link']
  
  UNIQUE (eventId, attendeeId)
}
```

### Indexes
- `events.slug` (unique)
- `events.date` (for upcoming events queries)
- `rsvps.eventId` (for attendee lists)
- `rsvps.attendeeId` (for user's RSVP history)
- `attendance.eventId` (for check-in stats)

## Environment Variables

Required in `.env` file (see `.env.example` for reference):

```env
# Session Secret (generate with: openssl rand -base64 32)
SESSION_SECRET=your_random_secret_key

# Database (choose one)
# Option 1: Cloudflare D1 (production)
DATABASE_ID=your_d1_database_id

# Option 2: Turso (alternative)
DATABASE_URL=libsql://your-database.turso.io
DATABASE_AUTH_TOKEN=your_turso_token

# Email Service (choose one)
# Option 1: Resend (recommended)
RESEND_API_KEY=re_your_key
FROM_EMAIL=events@fubarlabs.org

# Option 2: SendGrid
SENDGRID_API_KEY=SG.your_key
FROM_EMAIL=events@fubarlabs.org

# Admin Credentials (change in production!)
ADMIN_EMAIL=admin@fubarlabs.org
ADMIN_PASSWORD_HASH=your_bcrypt_hash

# Cloudflare (for deployment)
CLOUDFLARE_ACCOUNT_ID=your_account_id
CLOUDFLARE_API_TOKEN=your_api_token

# Optional
DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/...
```

## Deployment

### Cloudflare Pages Setup

1. **Install Wrangler CLI**:
   ```bash
   pnpm add -g wrangler
   wrangler login
   ```

2. **Create D1 Database**:
   ```bash
   wrangler d1 create fubar-rsvp
   # Copy the database_id to wrangler.toml
   ```

3. **Run Migrations**:
   ```bash
   pnpm db:migrate:prod
   ```

4. **Deploy**:
   ```bash
   pnpm deploy
   # Or connect GitHub repo for automatic deployments
   ```

5. **Set Environment Variables**:
   ```bash
   wrangler pages secret put SESSION_SECRET
   wrangler pages secret put RESEND_API_KEY
   wrangler pages secret put ADMIN_PASSWORD_HASH
   ```

### GitHub Integration

1. **Connect Repository**:
   - Go to Cloudflare Pages dashboard
   - Click "Create a project" → "Connect to Git"
   - Select your GitHub repository
   - Build settings:
     - Build command: `pnpm build`
     - Build output directory: `build/client`
     - Root directory: `/`

2. **Auto-Deployments**:
   - Push to `main` branch → Production deployment
   - Push to feature branches → Preview deployments
   - Pull request comments get preview URLs automatically

### Local Testing

1. **Start Dev Server**:
   ```bash
   pnpm dev
   # Runs on http://localhost:3000
   # Uses local SQLite database (dev.db)
   ```

2. **Test with Production Database**:
   ```bash
   pnpm dev:remote
   # Connects to Cloudflare D1 for testing
   ```

3. **Run E2E Tests**:
   ```bash
   pnpm test:e2e
   # Starts dev server, runs Playwright tests
   ```

## Troubleshooting

### Tests Failing
```bash
pnpm test -- --reporter=verbose    # See detailed failures
pnpm typecheck                      # Check for type errors
```

### Database Issues
```bash
pnpm db:studio                      # Open GUI to inspect data
wrangler d1 execute fubar-rsvp --command "SELECT * FROM events"
```

### Deployment Failures
```bash
wrangler pages deployment tail      # View real-time logs
pnpm build                          # Test build locally
```

### Git Conflicts
```bash
git status                          # View conflicts
# Resolve in editor, then:
git add <resolved-files>
git commit
```

## Security Considerations

- **Session Management**: HttpOnly cookies, SameSite=Lax
- **Password Hashing**: bcrypt with 10 rounds minimum
- **SQL Injection**: Drizzle ORM prevents this by default
- **XSS Protection**: React escapes by default, validate user input
- **CSRF Protection**: Remix CSRF tokens on all forms
- **Rate Limiting**: Cloudflare rate limiting on login/RSVP routes
- **Email Validation**: Validate + sanitize email addresses
- **Waiver Storage**: Store IP address and timestamp for legal compliance

## Accessibility Checklist

- ✅ All forms have proper labels
- ✅ Keyboard navigation works throughout
- ✅ Focus indicators visible
- ✅ ARIA labels on interactive elements
- ✅ Semantic HTML (nav, main, section, etc.)
- ✅ Color contrast meets WCAG AA
- ✅ Error messages announced to screen readers
- ✅ Success messages announced to screen readers
- ✅ Mobile viewport tested

## Performance Targets

- **Lighthouse Score**: >90 on all metrics
- **First Contentful Paint**: <1.5s
- **Time to Interactive**: <3s
- **Total Blocking Time**: <200ms
- **Cumulative Layout Shift**: <0.1

## Additional Resources

- [Remix Documentation](https://remix.run/docs)
- [Cloudflare Pages Documentation](https://developers.cloudflare.com/pages)
- [Drizzle ORM Documentation](https://orm.drizzle.team/docs/overview)
- [shadcn/ui Components](https://ui.shadcn.com/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Zod Validation](https://zod.dev/)
- [Playwright Testing](https://playwright.dev/)
- [Conventional Commits](https://www.conventionalcommits.org/)

## Future AI Integration

When ready to add Claude AI:
- Use Anthropic API directly (no framework needed)
- Create `/api/chat` route for streaming responses
- Add chat interface to event pages for Q&A
- Store conversation history in database
- Use Vercel AI SDK or LangChain if needed
- Consider Cloudflare AI Workers for embeddings

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ricklon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
