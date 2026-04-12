## eyetraining

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Job Management** is a Next.js 16 application using the App Router with TypeScript, Material UI (MUI), TypeORM, and NextAuth for authentication. The project uses either SQLite (default) or PostgreSQL for data persistence.

## Essential Commands

### Development

```bash
npm run dev                    # Start dev server (uses PORT and HOSTNAME from .env)
npm run build                  # Build for production
npm start                      # Start production server
```

### Code Quality (Run before commits!)

```bash
npm run check                  # Run all checks (format, lint, type-check)
npm run check:fix              # Auto-fix formatting and linting issues
npm run format                 # Format code with Prettier
npm run format:check           # Check if code is formatted
npm run lint                   # Check for linting errors
npm run lint:fix               # Fix auto-fixable linting errors
npm run type-check             # Check TypeScript types
```

### Database Migrations

```bash
npm run migration:run          # Run pending migrations
npm run migration:revert       # Revert last migration
npm run migration:show         # Show migration status
npm run migration:generate -- -n MigrationName    # Generate migration from entity changes
npm run migration:create -- src/lib/db/migrations/MigrationName    # Create empty migration
```

**IMPORTANT**: Database synchronization is **disabled**. Always use migrations for schema changes.

## Architecture Overview

### Technology Stack

- **Framework**: Next.js 16 (App Router)
- **Language**: TypeScript (strict mode)
- **UI Library**: Material UI (MUI) v7 - **No Tailwind CSS**
- **Database**: TypeORM with SQLite (default) or PostgreSQL
- **Authentication**: NextAuth v5 (beta) with JWT sessions
- **Validation**: Zod schemas
- **Code Quality**: ESLint + Prettier + TypeScript ESLint

### Key Architectural Patterns

#### 1. Database Layer (TypeORM)

- **Location**: `src/lib/db/`
- **Entities**: TypeORM entities in `entities/` (e.g., `User.ts`)
- **Migrations**: All schema changes in `migrations/` directory
- **Initialization**: Database auto-initializes on startup via `init.ts`
- **Default Admin**: Created automatically from environment variables
- **Configuration**: Supports both SQLite and PostgreSQL via `DATABASE_URL`

```typescript
// Database is initialized automatically, access via:
import { getDataSource } from '@/lib/db/init';

const dataSource = getDataSource();
const userRepo = dataSource.getRepository(User);
```

#### 2. Authentication (NextAuth)

- **Configuration**: `src/lib/auth.ts`
- **Provider**: Credentials-based (username/password with bcrypt)
- **Session**: JWT strategy
- **Middleware**: Protects `/dashboard/*` routes (`src/middleware.ts`)
- **User Roles**: `ADMIN` and `USER` (enum in `User` entity)

```typescript
// In server components or API routes:
import { auth } from '@/lib/auth';
const session = await auth();

// In client components:
import { useSession } from 'next-auth/react';
const { data: session } = useSession();
```

#### 3. Validation Layer

- **Location**: `src/lib/validations/`
- **Tool**: Zod schemas for all input validation
- **Pattern**: Define schema → infer TypeScript types
- **Usage**: Validate in API routes before processing

```typescript
// Example from lib/validations/user.ts:
const createUserSchema = z.object({
  /* ... */
});
type CreateUserInput = z.infer<typeof createUserSchema>;
```

#### 4. API Routes

- **Location**: `src/app/api/`
- **Structure**: RESTful with Next.js route handlers (`route.ts`)
- **Standard Pattern**:
  1. Authenticate (if needed)
  2. Parse request body
  3. Validate with Zod
  4. Process with TypeORM
  5. Return standardized response

```typescript
// Response format:
{ data: T } | { error: string, details?: unknown }
```

#### 5. Component Architecture

- **Location**: `src/components/`
- **Organization**:
  - `ui/` - Base reusable components (Button, Input, etc.)
  - `forms/` - Form-specific components
  - `features/` - Feature-specific components
  - `layouts/` - Layout components
- **Server vs Client**: Default to Server Components, use `"use client"` only when needed
- **Styling**: MUI `sx` prop (no Tailwind CSS)

#### 6. Theme System

- **Configuration**: `src/theme/theme.ts`
- **Provider**: `ThemeRegistry.tsx` wraps the app
- **Typography**: Uses Geist Sans font (Next.js font optimization)
- **Customization**: Centralized MUI theme configuration

### Folder Structure

```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API routes (route.ts files)
│   ├── dashboard/         # Protected dashboard pages
│   ├── signin/            # Sign-in page
│   ├── layout.tsx         # Root layout
│   ├── page.tsx           # Home page
│   └── globals.css        # Minimal global styles
├── components/            # React components
│   ├── ui/               # Base UI components
│   ├── forms/            # Form components
│   ├── features/         # Feature-specific components
│   └── layouts/          # Layout components
├── lib/                   # Business logic & utilities
│   ├── db/               # Database layer
│   │   ├── entities/     # TypeORM entities
│   │   ├── migrations/   # Database migrations
│   │   ├── repositories/ # Custom repositories
│   │   ├── config.ts     # TypeORM configuration
│   │   └── init.ts       # Database initialization
│   ├── validations/      # Zod schemas
│   ├── utils/            # Utility functions
│   ├── hooks/            # Custom React hooks
│   ├── constants/        # App constants
│   └── auth.ts           # NextAuth configuration
├── types/                 # TypeScript type definitions
├── theme/                 # MUI theme configuration
└── middleware.ts          # Next.js middleware (auth)
```

### Import Patterns

Always use path aliases (configured in `tsconfig.json`):

```typescript
// ✅ Good
import { Button } from '@/components/ui/Button';
import { formatDate } from '@/lib/utils/date';
import { User } from '@/lib/db/entities/User';

// ❌ Bad
import { Button } from '../../../components/ui/Button';
```

## Critical Development Standards

### Naming Conventions (Strictly Enforced)

- **Variables/Functions**: `camelCase` (e.g., `userData`, `getUserById()`)
- **Components/Classes/Interfaces**: `PascalCase` (e.g., `UserCard`, `User`)
- **Constants**: `SCREAMING_SNAKE_CASE` (e.g., `MAX_RETRIES`)
- **Files**: Components use `PascalCase.tsx`, utilities use `camelCase.ts`
- **Database**: Tables are `lowercase_plural`, columns are `snake_case`
- **TypeORM Entities**: Class in `PascalCase`, properties in `camelCase`

### TypeScript Requirements

- **Strict mode enabled**: No `any` types allowed (use `unknown` if needed)
- **Explicit return types**: Recommended for exported functions
- **No unused variables**: `noUnusedLocals` and `noUnusedParameters` enabled

### Security Considerations

- **Never** commit secrets (use `.env.local`, not tracked by git)
- **Always** validate user input with Zod schemas
- **Always** sanitize error messages (don't expose internals)
- **Use** parameterized queries (TypeORM handles this)
- **Hash** passwords with bcrypt (already implemented)

### Code Quality Rules

- **ESLint**: Strict rules with security and code quality plugins
- **Prettier**: Auto-formats code (single quotes, 2-space indent)
- **Import Sorting**: Automatic via Prettier plugin (React → Next.js → MUI → third-party → local)

## Database Workflow

### Modifying the Schema

1. **Update Entity** (e.g., `src/lib/db/entities/User.ts`):

```typescript
@Column({ nullable: true })
phoneNumber?: string;
```

2. **Generate Migration**:

```bash
npm run migration:generate -- -n AddPhoneNumber
```

3. **Review Generated Migration** in `src/lib/db/migrations/`

4. **Run Migration**:

```bash
npm run migration:run
```

5. **Update Validations** (e.g., `src/lib/validations/user.ts`)

### Creating New Entities

1. Create entity file in `src/lib/db/entities/`
2. Add to `entities` array in `src/lib/db/config.ts`
3. Generate and run migration
4. Create validation schemas
5. Build API routes if needed

## Testing & Verification

Before committing:

```bash
npm run check:fix    # Auto-fix issues
npm run type-check   # Verify types
npm run build        # Test production build
```

## Environment Setup

Copy `.env.example` to `.env.local` and configure:

- `DATABASE_URL`: Database connection (SQLite or PostgreSQL)
- `NEXTAUTH_SECRET`: Random secret for JWT signing
- `ADMIN_USERNAME`, `ADMIN_PASSWORD`, `ADMIN_EMAIL`: Default admin user

## Documentation

Comprehensive documentation in `docs/` folder:

- `naming-conventions.md` - Variable, function, file naming
- `folder-structure.md` - Project organization
- `typescript-standards.md` - TypeScript usage patterns
- `component-patterns.md` - React component guidelines
- `api-guidelines.md` - API route standards
- `database-standards.md` - TypeORM and migration patterns
- `styling-guide.md` - MUI styling conventions
- `code-review-checklist.md` - Review standards

Refer to these docs for detailed patterns and examples.

## Common Pitfalls to Avoid

1. **Don't modify existing migrations** - Always create new ones
2. **Don't use `synchronize: true`** - It's disabled for safety
3. **Don't skip validation** - Always use Zod schemas in API routes
4. **Don't use Tailwind CSS** - This project uses MUI exclusively
5. **Don't use barrel exports** (index files) for components
6. **Don't use relative imports** - Use path aliases (`@/`)
7. **Don't put business logic in components** - Keep in `lib/`

## Development Workflow

1. **Create feature branch**: `git checkout -b feature/description`
2. **Make changes** following all conventions
3. **Run checks**: `npm run check:fix && npm run type-check`
4. **Create migration** if schema changed
5. **Test locally**: `npm run dev` and verify functionality
6. **Commit** with clear message
7. **Create PR** and use `docs/code-review-checklist.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tholum)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tholum)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
