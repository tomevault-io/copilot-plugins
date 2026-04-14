## bolgetv

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Salam News Portal** is a full-stack news publishing platform with an admin dashboard. It's a monorepo containing:
- **Client**: React SPA with TypeScript (Vite + React Router via Wouter)
- **Server**: Express.js with TypeScript
- **Shared**: Zod schemas and types shared between client and server
- **Database**: PostgreSQL (Neon serverless) with Drizzle ORM

The platform allows editors to create/manage articles, categories, and media files; public users browse and read news organized by category.

## Common Development Commands

```bash
# Development
npm run dev                    # Start dev server (Port 3000: tsx watches server/index.ts, Vite watches client)

# Type checking
npm run check                  # TypeScript type checking (both client and server)

# Database
npm run db:push              # Push schema changes to database (Drizzle migrations)

# Build & Production
npm run build                # Build client (Vite) + bundle server (esbuild)
npm start                    # Run production build (requires NODE_ENV=production)
```

**Environment variables required**: `DATABASE_URL`, `JWT_SECRET`, `ADMIN_USERNAME`, `ADMIN_PASSWORD`, `NODE_ENV`, `PORT` (default 3000 in dev, 5000 in production)

## Architecture & Key Patterns

### Full-Stack Monorepo Structure
- **Path aliases**: `@/*` → `client/src/*`, `@shared/*` → `shared/*` (defined in tsconfig.json and vite.config.ts)
- **Single port model**: App serves both API (`/api/*`) and static frontend on one port (configured via `PORT` env var)
- **Build output**: Client builds to `dist/public/`, server bundles to `dist/index.js`
- **Internationalization**: i18next configured for multi-language support (see `client/src/providers/I18nProvider.tsx`)

### Frontend Architecture
- **Routing**: Wouter (lightweight URL-based routing, not React Router)
- **State Management**: React Context (AuthContext) + TanStack Query (React Query v5) for server state
- **UI Components**: Radix UI primitives with custom Tailwind styling (components in `client/src/components/ui/`)
- **Forms**: React Hook Form + Zod validation (using `@hookform/resolvers`)
- **Theme**: Dark/light mode via next-themes, persisted to localStorage
- **Styling**: Tailwind CSS with custom color system and design tokens (see `design_guidelines.md`)
- **Rich Text Editing**: React Quill for article content editing with image/video support
- **Icons**: Lucide React for consistent icon library
- **Animations**: Framer Motion for smooth transitions and interactive elements
- **Internationalization**: i18next with react-i18next for multi-language UI

### Backend Architecture
- **Auth**: JWT tokens (jsonwebtoken) + local strategy (Passport.js) for admin login
- **Request logging**: Middleware in `server/index.ts` logs all `/api/*` requests with method, path, status, duration
- **Error handling**: Global error handler in `server/index.ts` catches and returns errors with status codes
- **File uploads**: Multer middleware, files stored in `uploads/` directory, served via `/uploads/*`
- **Admin initialization**: Auto-creates admin user from `ADMIN_USERNAME`/`ADMIN_PASSWORD` env vars on startup

### Database Schema
Core tables in PostgreSQL (accessed via Drizzle):
- `users`: Admin accounts (id, username, password, role, createdAt)
- `categories`: Article categories (id, name, slug, description, color, isActive, createdAt)
- `articles`: Articles (id, title, slug, content, excerpt, featuredImage, videoUrl, authorId, categoryId, status, isFeatured, viewCount, publishedAt, etc.)
- `media`: Uploaded files (id, filename, filePath, fileSize, mimeType, uploadedBy, createdAt)
- `tags`: Article tags (id, name, slug, createdAt)
- `articleTags`: Junction table (articleId, tagId)

**Schema location**: `shared/schema.ts` (Drizzle table definitions + Zod insert schemas + TypeScript types)

### Key Dependencies
- **Express.js**: HTTP server
- **Drizzle ORM**: Type-safe database access
- **Neon serverless**: PostgreSQL with WebSocket support
- **React Hook Form**: Form state management
- **TanStack Query**: Server state management
- **Lucide React**: Icon library
- **Framer Motion**: Animations
- **React Quill**: Rich text editor for articles

## Important Files & Patterns

### Server
- `server/index.ts`: Express app setup, middleware, error handling, port configuration
- `server/routes.ts`: All API endpoints (auth, articles, categories, media, tags)
- `server/auth.ts`: JWT token generation/validation, password hashing (bcryptjs), admin initialization
- `server/storage.ts`: Database abstraction layer (queries/mutations for all entities)
- `server/middleware/auth.ts`: `requireAdmin` middleware for protected routes
- `server/utils/upload.ts`: Multer file upload handling
- `server/utils/slugify.ts`: Slug generation and uniqueness logic

### Client
- `client/src/App.tsx`: Wouter router setup, context providers (Auth, Theme, Query, Tooltip, i18n)
- `client/src/contexts/AuthContext.tsx`: Auth state, login/logout, token management
- `client/src/pages/`: Page components (HomePage, ArticlePage, TechCategoryPage, Admin pages)
- `client/src/pages/Admin*.tsx`: Admin dashboard pages (AdminDashboard, AdminArticles, AdminArticleEditor, AdminCategories, AdminMedia)
- `client/src/components/`: Reusable UI components (TopNav, ThemeProvider, RichTextEditor, etc.)
- `client/src/components/ui/`: Radix UI primitive components (auto-generated from shadcn/ui)
- `client/src/lib/queryClient.ts`: React Query client configuration
- `client/src/lib/utils.ts`: Utility functions (cn classname merger, etc.)
- `client/src/providers/I18nProvider.tsx`: i18next configuration and language provider

## Development Workflow

### Adding a New Feature
1. **Schema first**: Define tables/columns in `shared/schema.ts` if needed
2. **Database**: Run `npm run db:push` to apply migrations
3. **API endpoint**: Add route in `server/routes.ts`, likely using storage functions
4. **Client component**: Build UI in React/Tailwind, use TanStack Query for data fetching
5. **Type safety**: Leverage shared Zod schemas from `@shared/schema`

### Adding an Admin Feature
1. Create/edit page component in `client/src/pages/Admin*.tsx`
2. Wrap in `<ProtectedRoute>` in App.tsx router
3. Add corresponding API endpoint in `server/routes.ts` (POST/PUT/DELETE)
4. Use TanStack Query hooks for mutations (create, update, delete)
5. Validation via Zod schemas from `shared/schema.ts`

### Admin Dashboard Pages
- **AdminDashboard** (`/admin/dashboard`): Overview with statistics and recent activity
- **AdminArticles** (`/admin/articles`): List of articles with filtering and management options
- **AdminArticleEditor** (`/admin/articles/new`, `/admin/articles/:id/edit`): Create/edit articles with rich text editor
- **AdminCategories** (`/admin/categories`): Manage article categories with color customization
- **AdminMedia** (`/admin/media`): Media library with upload and file management capabilities

All admin pages require authentication via `<ProtectedRoute>` wrapper which redirects unauthenticated users to `/admin/login`.

### Styling Guidelines
Follow `design_guidelines.md`:
- Use Tailwind utility classes (spacing scale: 2, 3, 4, 6, 8, 12, 16, 20)
- Colors: primary (teal), accent (coral), background, surface, text-primary, text-secondary
- Typography: Playfair Display for headlines, Inter for body (via Tailwind)
- Responsive: Base (mobile), md (640px+), lg (1024px+), xl
- Dark mode: Automatic via next-themes, all components should support it

## Common Gotchas & Tips

1. **Port Configuration**: App only listens on port specified by `PORT` env var. Development defaults to 3000 (via package.json script), production defaults to 3000 if not specified. This is the single port handling both API and frontend.

2. **Database connection**: If `DATABASE_URL` is not set and `NODE_ENV !== 'development'`, the app crashes at startup. In dev mode, it logs a warning and uses mock db.

3. **Admin user initialization**: Always set `ADMIN_USERNAME`, `ADMIN_PASSWORD`, and `JWT_SECRET` env vars. Admin is auto-created on first run if it doesn't exist.

4. **Slug uniqueness**: Both categories and articles have unique slug constraints. Use `generateUniqueSlug()` utility when creating.

5. **File uploads**: Use `upload.ts` middleware with Multer; files go to `uploads/` directory. Remember to delete files when deleting media/articles.

6. **JWT tokens**: Valid for the session. Tokens are stored in localStorage on client, included in `Authorization: Bearer <token>` header.

7. **API logging**: All `/api/*` requests are logged to console with method, path, status code, duration, and (if <80 chars) the JSON response.

8. **Vite setup**: Client root is `client/` (not project root). Build outputs to `dist/public/`. Server bundles to `dist/index.js`.

9. **Replit integration**: Project uses Replit dev plugins (`@replit/vite-plugin-*`) for enhanced development experience. These include error overlay, cartographer (file navigation), and dev banner - automatically stripped in production builds.

## Type Checking & Compilation

- `npm run check`: Runs TypeScript compiler in no-emit mode (type checking only)
- Path aliases work in both client and server via shared `tsconfig.json` config
- Strict mode enabled; avoid `any` types, leverage Zod schemas for runtime validation

## Data Fetching & Server State Management

The app uses TanStack Query v5 for all server-state operations on the client:

**Query Patterns:**
- Define hooks in component files (e.g., `useArticles`, `useArticleBySlug`) using `useQuery`
- Cache invalidation: Use `queryClient.invalidateQueries()` after mutations to refetch data
- Key strategy: `[resourceType, filters/id]` e.g., `['articles', { category: 'tech' }]`

**Mutation Patterns:**
- Use `useMutation` with `onSuccess` to invalidate related queries
- Error handling: Catch and display via toast notifications
- Optimistic updates: Optional but recommended for UI responsiveness

**API Request Utility:**
- Use a centralized fetch utility (client-side) that auto-injects JWT tokens in `Authorization: Bearer <token>` header
- All API calls are typed against Zod schemas from `@shared/*`
- Responses validated at runtime via Zod parsing

## Database & Server Layer

### Database Connection
- **Development mode**: If `DATABASE_URL` is missing, a warning logs but app continues with mock DB
- **Production mode**: App crashes immediately if `DATABASE_URL` is not set; never attempt to run without it
- Connection uses Neon's WebSocket support via `@neondatabase/serverless`

### Server-Side Data Access
- `server/storage.ts`: Repository pattern implementation; all DB queries go through this interface
- `server/db.ts`: Database client configuration and connection pooling setup
- `server/seed.ts`: Optional utility for populating seed data in development/testing (not auto-run)

### Response Logging Detail
- All `/api/*` requests logged with `METHOD PATH STATUS_CODE ms :: JSON_RESPONSE`
- Response JSON only included if stringified length ≤ 80 characters; longer responses truncated with `…`
- Useful for debugging but be aware logs may be truncated for large payloads

## Directory Structure

```
.
├── client/                   # React SPA
│   └── src/
│       ├── pages/           # Page components
│       ├── components/      # Reusable components
│       │   └── ui/          # Radix UI primitives (shadcn/ui)
│       ├── contexts/        # React Context (Auth, Theme, etc.)
│       ├── hooks/           # Custom React hooks
│       ├── lib/             # Utilities, Query client
│       └── App.tsx          # Wouter router setup
├── server/                   # Express backend
│   ├── index.ts             # App entry point, middleware setup
│   ├── routes.ts            # All API endpoints (REST)
│   ├── auth.ts              # JWT & password hashing utilities
│   ├── storage.ts           # Database layer (IStorage interface)
│   ├── db.ts                # Drizzle client & connection config
│   ├── seed.ts              # Optional database seeding utility
│   ├── vite.ts              # Vite dev server integration
│   ├── middleware/
│   │   └── auth.ts          # requireAdmin middleware
│   └── utils/
│       ├── upload.ts        # Multer file upload handling
│       └── slugify.ts       # Slug generation & uniqueness
├── shared/                   # Shared types/schemas
│   └── schema.ts            # Drizzle tables + Zod insert schemas + TypeScript types
├── dist/                     # Build output (gitignored)
│   ├── public/              # Client build (Vite output)
│   └── index.js             # Server bundle (esbuild output)
├── migrations/              # Drizzle migration files (auto-generated by db:push)
├── uploads/                 # User-uploaded media files (gitignored)
├── vite.config.ts           # Vite client build config
├── drizzle.config.ts        # Drizzle migration config
├── tsconfig.json            # TypeScript compiler options + path aliases
├── package.json             # Project dependencies & npm scripts
└── design_guidelines.md     # UI/UX design system specs
```

## Development vs Production

### Development
- `npm run dev` runs both Vite (client) and tsx watch (server) in parallel
- `NODE_ENV=development` allows app to run without `DATABASE_URL` (mock DB mode)
- Automatic hot reload on file changes; errors overlay in browser
- Vite handles `/` serving and proxies `/api/*` to Express

### Production
- `npm run build` compiles client (Vite) to `dist/public/` and bundles server (esbuild) to `dist/index.js`
- `npm start` runs the production bundle with `NODE_ENV=production`
- **Critical**: All env vars must be set (DATABASE_URL, JWT_SECRET, ADMIN_USERNAME, ADMIN_PASSWORD, NODE_ENV)
- Static files served directly from Express; no Vite dev server
- Single process handles both API and frontend serving on configured port (default 5000)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nahidmursalzada-sketch)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nahidmursalzada-sketch)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
