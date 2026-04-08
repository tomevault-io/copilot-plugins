## sportshub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Note:** This file focuses on the Next.js dashboard application. For repository-level guidance including the Google Apps Script setup, see `../CLAUDE.md` in the parent directory.

## Project Overview

This is a full-stack Next.js dashboard for analyzing the 106th National Sports Festival (제106회 전국체육대회) results. The application provides:
- Public dashboard with regional rankings and score visualizations
- Detailed analysis pages (regions, sports, categories)
- Comparison tools for regions and sports performance
- Protected admin area for comprehensive data management (CRUD operations)
- Google Sheets as the database backend
- Real-time data visualization with Recharts

**Tech Stack**: Next.js 16 (App Router), TypeScript, Tailwind CSS 4, shadcn/ui, Zustand, NextAuth.js, Google Sheets API, Recharts, React Hook Form, Zod

## Development Commands

### Running the Application
```bash
# Install dependencies (first time setup)
npm install

# Start development server (http://localhost:3000)
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linter
npm run lint
```

### Working with shadcn/ui Components
```bash
# Add new UI components (e.g., badge, toast)
npx shadcn@latest add <component-name>
```

## Environment Setup

Create `.env.local` with these required variables:
```env
# Google Sheets API
GOOGLE_SPREADSHEET_ID=your_spreadsheet_id
GOOGLE_SERVICE_ACCOUNT_EMAIL=service-account@project.iam.gserviceaccount.com
GOOGLE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"

# Google OAuth (for admin login)
GOOGLE_CLIENT_ID=your_client_id
GOOGLE_CLIENT_SECRET=your_client_secret

# NextAuth
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=your_nextauth_secret

# Admin Access (comma-separated emails)
ADMIN_EMAILS=admin@example.com,admin2@example.com
```

**Note**: The app falls back to mock data if Google Sheets credentials are missing, allowing development without full API setup.

## Architecture Overview

### Data Flow
```
User Request
    ↓
Next.js Page (Server Component)
    ↓
API Route (/api/scores, /api/regions, /api/sports, /api/sport-events, /api/configs)
    ↓
Google Sheets API Wrapper (lib/google-sheets.ts)
    ↓
Google Sheets (Database)
```

### Key Architectural Patterns

1. **Server Components by Default**: Pages use async/await to fetch data server-side
2. **Client Components for Interactivity**: Forms, dialogs, and interactive UI use `'use client'` directive
3. **API Routes for Mutations**: All CRUD operations go through Next.js API routes
4. **Zustand for Client State**: Global UI state (filters, selections) managed with Zustand store
5. **Protected Routes**: Admin pages check session via NextAuth and redirect unauthenticated users
6. **Hierarchical Data Model**: Sport → SportEvent (세부종목) → Division (종별) → Scores

### Critical File Locations

| Purpose | Path |
|---------|------|
| Google Sheets integration | `src/lib/google-sheets.ts` |
| Authentication config | `src/lib/auth.ts` |
| Type definitions | `src/types/index.ts` |
| Record type calculations | `src/lib/record-types.ts` |
| Validation schemas (Zod) | `src/lib/validations.ts` |
| Global store (Zustand) | `src/store/index.ts` |
| Constants (sheet names, regions, divisions) | `src/lib/constants.ts` |
| Score management page | `src/app/admin/scores/page.tsx` |
| Score API endpoints | `src/app/api/scores/route.ts` |
| Dashboard homepage | `src/app/page.tsx` |
| Region comparison | `src/app/region-comparison/page.tsx` |
| Sports performance | `src/app/sports-performance/page.tsx` |

## Google Sheets Data Structure

The application expects these sheets in the Google Spreadsheet:

- **`regions`**: Region master data
  - Columns: `id`, `name`, `is_host`, `color`

- **`sports`**: Sports master data (e.g., track, field)
  - Columns: `id`, `name`, `sub_name`, `max_score`, `category`

- **`sport_events`**: Sub-events within each sport (e.g., track-남고-100m)
  - Columns: `id`, `sport_id`, `division`, `event_name`, `max_score`
  - Example: id="track-MHS-100m", sport_id="track", division="남고", event_name="100m"

- **`scores`**: Score data (종목별 × 종별 × 시도별)
  - Columns: `id`, `sport_id`, `sport_event_id`, `region_id`, `division`, `expected_rank`, `expected_score`, `expected_medal_score`, `actual_score`, `actual_medal_score`, `sub_event_total`, `converted_score`, `confirmed_bonus`, `record_type`, `total_score`, `gold`, `silver`, `bronze`, `rank`, `updated_at`
  - `division`: 남고, 여고, 고등, 남대, 여대, 대학, 남일, 여일, 일반 (9개)
  - Each combination of (sport_event_id, region_id, division) has one score record

- **`rank_score`**: Rank-based score configuration (순위별 점수 설정)
  - Columns: `id`, `sport_event_id`, `rank`, `acquired_score`, `medal_score`, `updated_at`
  - Defines scores for each rank (1st place, 2nd place, etc.) per sport event

- **`record_bonus`**: Record bonus multiplier configuration (신기록 가산 설정)
  - Columns: `id`, `sport_event_id`, `bonus_multiplier`, `updated_at`
  - Defines bonus multipliers for record-breaking performances

All sheets must have headers in the first row. The `google-sheets.ts` wrapper converts rows to objects using headers as keys.

## Core Types

```typescript
// Main entity types (src/types/index.ts)
Region: { id, name, is_host, color }

Sport: { id, name, sub_name?, max_score, category? }

SportEvent: { id, sport_id, division, event_name, max_score? }
// Example: { id: "track-MHS-100m", sport_id: "track", division: "남고", event_name: "100m" }

Score: {
  id, sport_id, sport_event_id?, region_id,
  division: '남고' | '여고' | '고등' | '남대' | '여대' | '대학' | '남일' | '여일' | '일반',
  expected_rank?, expected_score, expected_medal_score?,
  actual_score?, actual_medal_score?, sub_event_total?,
  converted_score?, confirmed_bonus?, record_type?,
  total_score?, gold?, silver?, bronze?, rank?, updated_at?
}

RankScoreConfig: { id, sport_event_id, rank, acquired_score, medal_score, updated_at? }

RecordBonusConfig: { id, sport_event_id, bonus_multiplier, updated_at? }
```

## Score Calculation Logic

The application implements a complex scoring system for sports events:

1. **Acquired Score (획득성적)**: Points earned based on rank in a specific event
2. **Medal Score (메달득점)**: Bonus points for medals (gold/silver/bronze)
3. **Converted Score (환산점수)**: `(획득성적 × 알점수) + 메달득점`
4. **Confirmed Bonus (확정자가산)**: `실제득점 × 신기록가산배율` for record-breaking performances
5. **Total Score (총 득점)**: Sum of all scores including bonuses

Record types (defined in `src/lib/record-types.ts`):
- 세계신기록 (World Record): 300% bonus
- 한국신기록 (Korean Record): 200% bonus
- 대회신기록 (Meet Record): 50% bonus
- And more...

## Component Organization

### shadcn/ui Components (`src/components/ui/`)
Pre-built, customizable UI primitives (button, card, dialog, table, etc.). **Do not manually edit** - regenerate using the shadcn CLI if changes are needed.

### Admin Components (`src/components/admin/`)
- **DataTable**: Reusable table with sorting, filtering, pagination
- **ExpandableDataTable**: Table with expandable rows for nested data
- **ScoreActions**: Row action menu (edit/delete) for scores
- **CreateScoreDialog**: Modal for creating new scores
- **QuickScoreDialog**: Quick entry dialog for score input
- **AdminSidebar**: Navigation sidebar for admin area
- **forms/**: Form components using React Hook Form + Zod (ScoreForm, SportForm, RegionForm, SportEventForm)
- **settings/**: Settings page components (ScoreConfigCard, RankScoreConfigCard, RecordBonusCard)

### Dashboard Components (`src/components/dashboard/`)
- **DashboardClient**: Main dashboard client component
- **RegionScoreCards**: Display top region scores with ranking badges

### Charts (`src/components/charts/`)
- **RegionRankingChart**: Horizontal bar chart using Recharts

### Comparison Components (`src/components/region-comparison/`, `src/components/sports-performance/`)
- **RegionComparisonClient**: Client component for region comparison
- **SportsPerformanceClient**: Client component for sports performance analysis

### Layout Components (`src/components/layout/`)
- **Header**: Global navigation with auth status
- **Footer**: Global footer

## Page Structure

### Public Pages
- `/` - Main dashboard with regional rankings and visualizations
- `/analysis/regions` - Regional analysis page
- `/analysis/sports` - Sports analysis page
- `/analysis/categories` - Category analysis page
- `/compare` - Comparison page
- `/region-comparison` - Region comparison page
- `/sports-performance` - Sports performance page
- `/login` - Login page

### Protected Admin Pages (require authentication)
- `/admin` - Admin dashboard
- `/admin/scores` - Score management (CRUD)
- `/admin/sports` - Sport management (CRUD)
- `/admin/regions` - Region management (CRUD)
- `/admin/settings` - System settings (rank scores, record bonuses)

## Working with Forms

All forms use **React Hook Form** + **Zod** for validation:

1. Define schema in `src/lib/validations.ts`
2. Use `useForm` with `zodResolver`
3. Handle submission in form component
4. Send validated data to API route

Example pattern:
```typescript
const form = useForm<ScoreFormData>({
  resolver: zodResolver(scoreSchema),
  defaultValues: { ... }
});

const onSubmit = async (data: ScoreFormData) => {
  const response = await fetch('/api/scores', {
    method: 'POST',
    body: JSON.stringify(data)
  });
  // Handle response
};
```

## Authentication & Authorization

- **Public routes**: `/`, `/analysis/*`, `/compare`, `/region-comparison`, `/sports-performance`
- **Protected routes**: `/admin/*` (requires authenticated session)
- **Admin check**: Email must be in `ADMIN_EMAILS` environment variable
- **Auth provider**: `NextAuthProvider` wraps the app in `src/app/layout.tsx`
- **Session access**: Use `useSession()` hook (client) or `getServerSession()` (server)

Protected pages redirect to login if unauthenticated:
```typescript
const session = await getServerSession(authOptions);
if (!session) {
  redirect('/api/auth/signin');
}
```

## API Endpoints

### Implemented APIs

**Scores:**
- `GET /api/scores` - List with filtering, sorting, pagination
- `POST /api/scores` - Create new score
- `GET /api/scores/:id` - Get single score
- `PUT /api/scores/:id` - Update score
- `DELETE /api/scores/:id` - Delete score
- `POST /api/scores/calculate-converted` - Calculate converted score

**Regions:**
- `GET /api/regions` - List all regions
- `POST /api/regions` - Create region
- `GET /api/regions/:id` - Get single region
- `PUT /api/regions/:id` - Update region
- `DELETE /api/regions/:id` - Delete region

**Sports:**
- `GET /api/sports` - List all sports
- `POST /api/sports` - Create sport
- `GET /api/sports/:id` - Get single sport
- `PUT /api/sports/:id` - Update sport
- `DELETE /api/sports/:id` - Delete sport

**Sport Events:**
- `GET /api/sport-events` - List sport events
- `POST /api/sport-events` - Create sport event
- `GET /api/sport-events/:id` - Get single sport event
- `PUT /api/sport-events/:id` - Update sport event
- `DELETE /api/sport-events/:id` - Delete sport event

**Configs:**
- `GET /api/configs/rank-score` - Get rank score configs
- `POST /api/configs/rank-score` - Create rank score config
- `POST /api/configs/rank-score/bulk` - Bulk update rank score configs
- `GET /api/configs/record-bonus` - Get record bonus configs
- `POST /api/configs/record-bonus` - Create record bonus config

**Admin:**
- `GET /api/admin/dashboard-data` - Get admin dashboard summary data

**Auth:**
- `POST /api/auth/change-password` - Change user password

All mutation endpoints (POST/PUT/DELETE) must:
1. Check authentication via `getServerSession()`
2. Validate input with Zod schema
3. Return appropriate status codes and error messages

## Styling Guidelines

- Use **Tailwind CSS utility classes** for all styling
- Use **shadcn/ui components** for consistent UI patterns
- Responsive design: Use `sm:`, `md:`, `lg:` breakpoints
- Colors: Use semantic tokens (`bg-primary`, `text-destructive`, etc.)
- Dark mode ready: Components use CSS variables for theming

## Zustand Store Structure

Global store (`src/store/index.ts`) manages:
- **Data**: `regions`, `sports`, `scores`, `categoryScores`
- **UI State**: `selectedRegionId`, `selectedSportId`, `selectedCategory`
- **Loading/Error**: `isLoading`, `error`
- **Actions**: `setRegions`, `setScores`, `fetchData`, etc.

Use store in components:
```typescript
const { scores, selectedRegionId, setSelectedRegionId } = useDashboardStore();
```

## Common Patterns

### Reading from Google Sheets
```typescript
import { getSheetData } from '@/lib/google-sheets';
const data = await getSheetData('scores');
```

### Writing to Google Sheets
```typescript
import { appendSheetData, updateSheetData, deleteSheetRow } from '@/lib/google-sheets';
await appendSheetData('scores', [newScore]);
await updateSheetData('scores', rowIndex, updatedScore);
await deleteSheetRow('scores', rowIndex);
```

### Using Constants
```typescript
import { REGIONS, SPORT_CATEGORIES, SHEET_NAMES, DIVISIONS, DIVISION_LIST } from '@/lib/constants';
```

### Parsing Numbers Safely
```typescript
import { parseNumber } from '@/lib/utils';
const score = parseNumber(value, 0); // Returns 0 if value is invalid
```

### Calculating Record Bonuses
```typescript
import { calculateConfirmedBonus, getRecordBonusPercentage } from '@/lib/record-types';
const bonus = calculateConfirmedBonus(actualScore, recordTypeId);
```

## Data Hierarchy

The application follows a hierarchical data structure:

```
Sport (종목)
  └── SportEvent (세부종목)
        └── Division (종별)
              └── Region (시도)
                    └── Score (성적)
```

Example:
- Sport: "육상 트랙" (track)
  - SportEvent: "100m 남고" (track-MHS-100m)
    - Division: "남고"
      - Region: "경기도", "부산", etc.
        - Score: Individual performance data per region

## Korean Localization

All UI text is in Korean. Error messages, labels, and validation messages are Korean by default. When adding new features, maintain Korean localization:
- Form labels: "종목", "시도", "점수", "종별", "세부종목" etc.
- Validation errors: Use Korean messages in Zod schemas
- Success/error toasts: Korean text

## Performance Considerations

- Google Sheets API has rate limits - cache responses when possible
- Use Next.js Image component for optimized images
- Leverage Server Components to reduce client JS bundle
- Consider implementing SWR or React Query for client-side data fetching if performance issues arise

## Known Limitations

1. **No database transactions**: Google Sheets doesn't support transactions
2. **Limited concurrent writes**: Multiple simultaneous writes may cause conflicts
3. **Mock data fallback**: Some features use mock data if Google Sheets credentials are not configured
4. **No real-time updates**: Users must refresh to see changes made by others

## Deployment

The project is designed for **Vercel** deployment:
1. Connect GitHub repo to Vercel
2. Set environment variables in Vercel dashboard
3. Deploy (automatic on push to main)

**Important**: Ensure all environment variables are set in Vercel, especially Google Cloud credentials and `ADMIN_EMAILS`.

## Testing Strategy

Currently, there are no automated tests. When adding tests:
- Use Jest/Vitest for unit tests
- Use React Testing Library for component tests
- Test API routes with mock Google Sheets responses
- Test form validation with Zod schemas
- Test score calculation logic in `record-types.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/forzamin521-a11y)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/forzamin521-a11y)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
