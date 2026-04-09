## aerobea-timeline

> Aerobea Timeline is a Next.js application that visualizes fictional political timelines for imaginary countries. It displays interactive timelines showing presidents/leaders and monarchs with their terms, lifespans, and party affiliations. The app features AI-powered generation of leader names and portraits.

# CLAUDE.md - Aerobea Timeline Project Context

## Project Overview

Aerobea Timeline is a Next.js application that visualizes fictional political timelines for imaginary countries. It displays interactive timelines showing presidents/leaders and monarchs with their terms, lifespans, and party affiliations. The app features AI-powered generation of leader names and portraits.

## Tech Stack

- **Framework**: Next.js 16 with App Router (React 19, TypeScript 5.9)
- **Database**: PostgreSQL with Prisma 6.19 ORM (+ optional Prisma Accelerate)
- **AI Services**: OpenAI GPT (text generation), FAL AI (image generation/editing)
- **Storage**: Vercel Blob for image uploads
- **Package Manager**: pnpm

## Project Structure

```
app/                    # Next.js App Router
├── api/               # REST API routes
│   ├── [country]/     # Dynamic country endpoints
│   ├── admin/         # Admin CRUD endpoints
│   ├── ai/            # AI generation endpoints (generate, edit, review)
│   ├── bootstrap/     # Database initialization
│   └── upload/        # File upload to Vercel Blob
├── admin/             # Admin dashboard UI
├── fog/               # Alternative "fog of war" timeline view
├── page.tsx           # Main timeline visualization
└── layout.tsx         # Root layout
components/            # React components
├── TimelineGrid.tsx   # Grid of president cards
├── PresidentCard.tsx  # Individual president card with visual bars
├── ProgressBar.tsx    # Interactive timeline scrubber
└── MusicControls.tsx  # Web Audio synthesizer controls
lib/                   # Utilities and types
├── timeline.ts        # Type definitions and helpers
├── prisma.ts          # Prisma client singleton
├── serialize.ts       # JSON serialization utilities
└── adminValidation.ts # Input validation for admin endpoints
data/                  # Static bootstrap data
├── bootstrapCountries.ts  # Pre-populated country data
├── exitReasons.ts     # Presidential exit scenarios
└── deathReasons.ts    # Death cause descriptions
prisma/                # Database schema and migrations
scripts/               # Utility scripts (migration, testing)
```

## Key Commands

```bash
pnpm dev              # Start development server (localhost:3000)
pnpm build            # Generate Prisma types + build for production
pnpm start            # Run production build
npx prisma generate   # Generate Prisma client types
npx prisma migrate dev # Run database migrations
```

## Environment Variables Required

```
DATABASE_URL          # PostgreSQL connection string
OPENAI_API_KEY        # For GPT text generation
FAL_API_KEY           # For AI image generation
VERCEL_BLOB_TOKEN     # For file storage (optional for local dev)
```

## Core Data Types (lib/timeline.ts)

```typescript
interface President {
  id: string;
  name: string;
  party: string;
  birthDate: Date;
  deathDate?: Date;
  deathCause?: string;
  termStart: Date;
  termEnd?: Date;
  exitReason?: string;
  imageUrl?: string;
}

interface Monarch {
  id: string;
  name: string;
  birthDate: Date;
  deathDate?: Date;
  deathCause?: string;
  reignStart: Date;
  reignEnd?: Date;
}

interface Country {
  id: string;
  code: string;
  name: string;
  presidents: President[];
  monarchs: Monarch[];
  startYear: number;
  endYear: number;
}
```

## Key API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/api/countries` | GET | Fetch all countries with presidents & monarchs |
| `/api/[country]/presidents` | GET | Fetch presidents for a country |
| `/api/admin/countries` | POST | Create new country |
| `/api/admin/countries/[id]` | PUT | Update country |
| `/api/ai/generate` | POST | Generate president images via FAL AI |
| `/api/bootstrap` | POST | Initialize database with bootstrap data |

## Coding Conventions

### TypeScript
- Strict mode enabled - all types must be explicit
- Use type guards (`isPresident()`, `isObject()`) for runtime validation
- Interfaces preferred over type aliases for object shapes

### React Patterns
- Use `'use client'` directive for interactive components
- Server Components by default in App Router
- Controlled components with useState for form inputs
- useCallback for memoized event handlers passed as props

### Database
- Prisma client singleton in `lib/prisma.ts` (avoids connection leaks)
- Presidents and monarchs stored as JSON in PostgreSQL
- Always run serialization/normalization on data load

### Validation
- Admin input validation in `lib/adminValidation.ts`
- Multi-level validation: type checks → required fields → date parsing → event validation
- Return detailed error messages for each invalid field

### Styling
- CSS Grid for card layouts: `grid-template-columns: repeat(auto-fill, minmax(150px, 1fr))`
- State-based borders: blue (alive), green (president), red (dead), dimmed (unborn)
- Visual bars in PresidentCard: pink (pre-birth), red (lifespan), green (term)

## Timeline Visualization Logic

The timeline plays back at ~90x speed (1 day per 4ms). Each PresidentCard shows:
- **Pink bar** (right edge): Days until birth
- **Red bar**: Remaining lifespan
- **Green bar**: Days remaining in current presidency
- **Image overlay**: Gradient based on alive/dead/president status

## Important Patterns

### Date Handling
Dates are stored as ISO strings in JSON but converted to Date objects when used:
```typescript
// Serialization (lib/serialize.ts)
president.birthDate = new Date(data.birthDate);
```

### API Response Format
```typescript
return NextResponse.json({ data }, { status: 200 });
return NextResponse.json({ error: message }, { status: 400 });
```

### Prisma Error Handling
```typescript
if (error instanceof Prisma.PrismaClientKnownRequestError) {
  if (error.code === 'P2002') // Unique constraint violation
  if (error.code === 'P2025') // Record not found
}
```

## Admin Features

- **JSON Export**: Copy entire country data via admin dashboard
- **Live Editing**: Inline edit presidents/monarchs with code editor
- **Validation**: Strict schema enforcement before database save
- **Party Colors**: Manage party → color mappings

## Testing

No test suite currently implemented. Manual testing via:
- Scripts in `/scripts/` for API testing (test-fal.ts, test-upload.ts)
- Bootstrap endpoint for database initialization
- Admin interface for data validation

## Notes for Development

1. **Image URLs**: Remote images must match pattern in `next.config.mjs` (currently `*.public.blob.vercel-storage.com`)
2. **Hash Routing**: Country selection persisted via URL hash (`#countrycode`)
3. **Web Audio**: MusicControls uses AudioContext - only works after user interaction
4. **Prisma Accelerate**: Optional caching layer - may need adjustment for local dev

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rewbs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rewbs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
