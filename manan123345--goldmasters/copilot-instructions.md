## goldmasters

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Goldmasters Spot-the-Ball Competition Platform - A password-protected spot-the-ball competition platform where participants purchase tickets to place markers on images (e.g., cricket scenes with ball removed). External judges determine winning coordinates, and the system automatically computes winners based on proximity.

## Technology Stack

- **Monorepo**: PNPM workspaces with 3 packages:
  - `apps/web`: Next.js 16 frontend (App Router, React 19, TypeScript)
  - `apps/api`: Express backend API (TypeScript, Node.js)
  - `packages/db`: Prisma ORM with PostgreSQL
- **Frontend**: Tailwind CSS, shadcn/ui, Framer Motion, Konva.js (canvas manipulation)
- **Backend**: Express, JWT auth, bcrypt password hashing
- **Database**: PostgreSQL with Prisma ORM
- **Deployment**: Vercel (frontend), AWS S3 (storage)

## Common Commands

### Development
```bash
# Start all apps in development mode
pnpm dev

# Start individual apps
pnpm dev:web        # Frontend only (localhost:3000)
pnpm dev:api        # Backend only (localhost:4000)
```

### Database Operations
```bash
# Generate Prisma client (run after schema changes)
pnpm db:generate

# Push schema changes to database (no migration files)
pnpm db:push

# Create and run migrations
pnpm db:migrate

# Open Prisma Studio (database GUI)
pnpm --filter db studio

# Seed the database
pnpm --filter db seed
```

### Building and Testing
```bash
# Build all apps for production
pnpm build

# Build individual apps
pnpm build:web
pnpm build:api
pnpm build:db       # Generates Prisma client and compiles types

# Run tests across all packages
pnpm test

# Lint all packages
pnpm lint
```

### Clean Commands
```bash
# Clean all build artifacts and node_modules
pnpm clean
```

## Architecture Overview

### Competition Flow
1. **DRAFT**: Admin creates competition with image, ticket limits, and password
2. **ACTIVE**: Competition opens for participant registration
3. **CLOSED**: Entry period ends (manual or when tickets sell out)
4. **JUDGING**: External judges review, admin enters judge coordinates
5. **COMPLETED**: System calculates winners by proximity, publishes results

### Data Model (Prisma Schema)

Core entities follow this relationship:
```
Competition (1) → (N) Participants
Competition (1) → (N) Tickets → (N) Markers
Competition (1) → (N) JudgeMarks → (1) Final Coordinates
Participants (N) → (1) Result (based on closest marker)
```

Key models:
- **Competition**: Holds image, ticket config, status, judging coordinates
- **Participant**: User entry with name/email/phone, ticket count
- **Ticket**: Sequential tickets with marker allowance (default: 3 markers/ticket)
- **Marker**: User-placed coordinates (x, y) on competition image
- **JudgeMark**: Individual judge coordinates (4 judges → averaged)
- **Result**: Calculated winner rankings by proximity
- **Winner**: Top N participants with prizes
- **AuditLog**: Full audit trail of all actions

### API Structure (`apps/api/src`)

Express REST API with modular routes:
- `routes/auth.routes.ts`: Admin JWT authentication (login)
- `routes/admin.routes.ts`: Admin dashboard, competition management, judging
- `routes/competition.routes.ts`: Public competition info, password verification
- `routes/participant.routes.ts`: Participant registration, marker placement

Middleware:
- `middleware/auth.middleware.ts`: JWT verification for protected routes
- CORS, helmet, compression configured in `index.ts`

### Frontend Structure (`apps/web/src`)

Next.js App Router structure:
- `app/page.tsx`: Landing page
- `app/competition/[id]/`: Competition entry flow (password, registration, marker placement)
- `app/admin/`: Admin dashboard (competitions, judging, results)
- `app/api/`: Next.js API routes (proxy to Express backend)
- `components/`: Reusable UI components (shadcn/ui)
- `server/`: Server-side utilities and actions
- `lib/`: Client utilities (Konva canvas helpers, API clients)

### Marker Placement (Konva.js)

The core UX feature uses react-konva for canvas marker placement:
- Load competition image onto canvas
- Allow 3 marker placements per ticket (configurable)
- Store normalized coordinates (0-1 range) for responsive rendering
- Calculate distance to judge coordinates using Euclidean distance

### Authentication & Security

- **Admin**: JWT-based auth with bcrypt password hashing
- **Participants**: Password-protected competition access (per-competition password)
- **API Security**: Helmet, CORS, input validation with express-validator
- **Database**: All passwords hashed with bcrypt (10 salt rounds)

## Development Notes

### Prisma Workflow
1. Modify `packages/db/prisma/schema.prisma`
2. Run `pnpm db:generate` to update Prisma client
3. Run `pnpm db:push` (dev) or `pnpm db:migrate` (prod) to update database
4. Import `@prisma/client` from `db` workspace package

### Environment Variables
Each app requires `.env` file:
- `apps/api/.env`: DATABASE_URL, JWT_SECRET, AWS_S3_CONFIG
- `apps/web/.env.local`: NEXT_PUBLIC_API_URL
- `packages/db/.env`: DATABASE_URL

### Monorepo Dependencies
- Apps import shared `db` package: `import { prisma } from 'db'`
- Changes to `packages/db` require rebuild: `pnpm build:db`
- PNPM workspace protocol: `"db": "workspace:*"` in package.json

### Testing Workflow
1. Create test competition via admin dashboard
2. Note competition password
3. Access competition URL as participant
4. Place markers using canvas
5. Enter judge coordinates in admin panel
6. System auto-calculates winners

## Deployment

### Vercel (Frontend)
- Connected to main branch for auto-deploy
- Build command: `pnpm install --no-frozen-lockfile && pnpm -w run db:generate && pnpm run build`
- Environment variables configured in Vercel dashboard
- API routes proxied to backend via `vercel.json` rewrites

### Backend API
- Deploy to AWS EC2 or Render
- Requires DATABASE_URL, JWT_SECRET, AWS credentials
- Run: `pnpm install && pnpm build && pnpm start`

### Database
- PostgreSQL on AWS RDS or Supabase
- Run migrations before deploy: `pnpm --filter db migrate:deploy`

## Important Constraints

- Node.js >= 20.10.0 required (exact version in web app)
- PNPM >= 8 required (uses workspace protocol)
- PostgreSQL database (Prisma schema uses postgresql provider)
- Canvas manipulation requires client-side rendering (Konva.js)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Manan123345)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Manan123345)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
