## progress-dashboard

> A React-based dashboard application that displays user progress for Git Mastery exercises. The app fetches exercise data and user progress from GitHub repositories and renders it in an interactive dashboard.

## Project Overview

A React-based dashboard application that displays user progress for Git Mastery exercises. The app fetches exercise data and user progress from GitHub repositories and renders it in an interactive dashboard.

**Key Features:**
- View user progress for Git Mastery exercises
- Fetch data from GitHub raw content (exercises and user progress)
- Display exercise status with visual indicators
- Real-time data refresh capability

## Tech Stack

### Core Technologies
- **React 19.0.0** - UI library
- **TypeScript 5.7.2** - Type safety
- **Vite 6.2.0** - Build tool and dev server
- **React Router 7.2.0** - Client-side routing (HashRouter)

### State Management & Data Fetching
- **React Query 3.39.3** - Server state management and caching
  - All API calls are wrapped in React Query hooks
  - Query keys follow pattern: `["query-name", ...dependencies]`
  - Queries are invalidated for manual data refresh

### Styling
- **Tailwind CSS 4.0.9** - Utility-first CSS framework
- **@tailwindcss/vite** - Vite integration for Tailwind
- Inline utility classes for all styling (no separate CSS modules)

### Utilities
- **axios 1.8.1** - HTTP client for API calls
- **react-icons 5.5.0** - Icon library (IoArrowBack, MdOutlineRefresh, etc.)

## Project Structure
```
src/
├── api/
│   └── queries/          # React Query hooks and API functions
├── components/
├── constants/           # App-wide constants
├── pages/              # Route-level page components
├── styles/             # Global styles (index.css)
└── types/              # TypeScript type definitions
```

## Common Tasks

### Adding a New Component
1. Create in appropriate folder in `components`, or a new folder in `components` if it does not fit the current folders
2. Use PascalCase for filename
3. Define props interface
4. Export `ui/` components as named export, and other components via an index.ts file within the subfolder
5. Style with Tailwind utilities
Example code: 
[src/components/dashboard/DashboardHeader.tsx](src/components/dashboard/DashboardHeader.tsx) - Shows component composition, Tailwind styling, and proper typing
[src/components/dashboard/index.ts](src/components/dashboard/index.ts) - Pattern for clean component exports

### Adding a New Page
1. Create in `pages/` matching URL structure
2. Use folder for route params: `(paramName)`
3. Add route to `main.tsx` Routes
4. Fetch data (if needed) with React Query hooks at page level

### Adding a New Query
1. Create file in `src/api/queries/` with snake_case name
2. Define response interface
3. Create async fetch function with axios
4. Export React Query hook with proper typing
5. Use `enabled` option if query depends on other data
Example code: 
[src/api/queries/get_user_progress.ts](src/api/queries/get_user_progress.ts) - Shows proper error handling, typing, and hook structure
[src/api/queries/get_exercises.ts](src/api/queries/get_exercises.ts) - Shows data transformation and sorting logic

## API Documentation

### Data Sources
- **Exercises directory**: `https://git-mastery.org/exercises-directory/exercises.json`
  - Returns: `Record<string, ExerciseRaw>` with lesson/detour info
- **User progress**: `https://raw.githubusercontent.com/{username}/{username}-gitmastery-progress/refs/heads/main/progress.json`
  - Returns: `UserProgress[]` with exercise_name and status
- **GitHub user API**: Standard GitHub REST API for user information

### API Client
- Use `axios` for all HTTP requests (already configured)
- Always return `null` on error (never throw)
- Wrap in React Query hooks with proper `enabled` conditions

### Error Handling
- API functions return `null` on error (no throws)
- Components handle `null` data gracefully
- Use optional chaining and nullish coalescing
- Loading states managed via React Query's `isLoading` and `isFetching`

### Data Refresh
- Manual refresh via `queryClient.invalidateQueries()`
- Refresh button calls invalidate for specific query keys

## Coding Conventions

### File Naming
- **Components**: PascalCase (`DashboardHeader.tsx`)
- **Utilities/Queries**: snake_case (`get_user_progress.ts`)
- **Types**: snake_case (`exercise.ts`)
- **Constants**: snake_case (`dashboard.ts`)

### Import Aliases
- `@/` maps to `src/` directory
- Always use path alias for src imports: `import { foo } from "@/api/queries/get_foo"`
- Configured in `vite.config.ts`

### Styling Conventions

**Tailwind Usage:**
- All styles via Tailwind utility classes (no custom CSS except global styles)
- Common patterns:
  - Flex layouts: `flex flex-row gap-2 items-center`
  - Colors: `text-gray-500`, `bg-blue-600`
  - Spacing: `mb-8`, `p-4`, `gap-2`
  - Text: `text-sm`, `text-xl font-bold`
- Extract repeated className strings to constants for consistency

**Responsive Design:**
- Mobile-first approach
- Use responsive prefixes as needed: `md:`, `lg:`

### Constants

- Centralize all magic strings/numbers in `constants/`
- Use `as const` for const objects to get literal types
- Export from barrel file for clean imports

### Routing

- Uses HashRouter for GitHub Pages compatibility
- Base path configured: `"/progress-dashboard/"`
- Route parameters accessed via `useParams()` hook
- Navigation via `Link` component from `react-router`

## Build & Development

### Commands

```bash
# Full build
npm run build

# Project-wide lint
npm run lint

# Start dev server
npm run dev
```

---
> Source: [git-mastery/progress-dashboard](https://github.com/git-mastery/progress-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-05 -->
