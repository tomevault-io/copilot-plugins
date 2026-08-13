## dellekeshub-nuxt-nest

> This is a monorepo with a **Nuxt 4 frontend** (Vue 3) and a **NestJS backend**.

# Agent Guidelines for DellekesHub

This is a monorepo with a **Nuxt 4 frontend** (Vue 3) and a **NestJS backend**.

---

## 1. Build / Lint / Test Commands

### Backend (NestJS)

```bash
# Navigate to backend
cd backend

# Install dependencies
npm install

# Build
npm run build

# Start in production
npm run start:prod

# Start in development (watch mode)
npm run start:dev

# Lint with ESLint (fixes issues)
npm run lint

# Format with Prettier
npm run format

# Create / update better-auth DB tables (user, session, account, verification)
npm run auth:migrate

# Generate a TypeORM SQL migration (production)
npm run db:generate -- src/migrations/<Name>

# Apply TypeORM migrations (production)
npm run db:migrate
```

### Frontend (Nuxt 4)

```bash
# Navigate to frontend
cd frontend

# Install dependencies
npm install

# Build for production
npm run build

# Start development server
npm run dev

# Generate static site
npm run generate

# Preview production build
npm run preview
```

---

## 2. Code Style Guidelines

### General

- **Project Type**: TypeScript is used in both frontend and backend
- **No comments**: Do not add code comments unless explicitly required
- **Error handling**: Use try/catch with proper error logging
- **Logging**: Use NestJS Logger for backend services (`private readonly logger = new Logger(ServiceName.name)`)

---

### Backend (NestJS)

#### Imports

- Use absolute imports with relative paths (e.g., `import { UsersService } from '../users/users.service'`)
- Group imports: external packages first, then internal modules
- Use named exports for services, controllers, modules

#### Naming Conventions

- **Files**: `kebab-case` (e.g., `auth.service.ts`)
- **Classes**: `PascalCase` (e.g., `AuthService`)
- **Methods/Properties**: `camelCase` (e.g., `findByUsername`)
- **Constants**: `UPPER_SNAKE_CASE`
- **Interfaces/Types**: `PascalCase` (e.g., `CreateVideo`)

#### TypeScript

- Enable strict mode in tsconfig where possible
- Use interfaces for object shapes, types for unions/intersections
- Use `any` sparingly (ESLint allows it but avoid when possible)
- Use Zod for runtime validation (already integrated)

#### Error Handling

- Use NestJS built-in exceptions: `BadRequestException`, `ConflictException`, `NotFoundException`, `UnauthorizedException`
- Throw exceptions with descriptive messages
- Use guards for authorization (see `auth/middleware/`)

#### REST API

- Follow RESTful conventions
- Validate request bodies with Zod schemas via `createZodValidationPipe(Schema)` on `@Body(...)`
- Validation schemas live in per-module `validation/` folders; reuse `LocalizedStringSchema` / `PaginationSchema` from `lib/`
- Return appropriate HTTP status codes

#### Testing

- No test suite is maintained in this repository. Do not add tests unless explicitly requested.

---

### Frontend (Nuxt 4 / Vue 3)

#### Structure

- Pages: `app/pages/`
- Components: `app/components/custom/` (business logic), `app/components/ui/` (shadcn-vue components)
- Composables: Use Nuxt `useFetch`, `useRouter`, `useRoute`
- Layouts: `app/layouts/`

#### Vue Components

- Use `<script setup>` syntax
- Use Composition API exclusively
- Use `ref` for primitives, `reactive` for objects
- Auto-imports are enabled (no import statements needed for composables)

#### Styling

- Use Tailwind CSS (v4)
- Use `tailwind-merge` and `clsx` for conditional classes
- shadcn-vue components in `components/ui/`
- Icons: Use `@nuxt/icon` module with Icon component

#### Data Fetching

- Use `useFetch()` for API calls
- Handle errors with try/catch
- Use loading states appropriately

#### Naming

- Components: `PascalCase` (e.g., `Navbar.vue`)
- Props: `camelCase`
- Events: `kebab-case` in templates

---

### ESLint & Prettier (Backend)

- ESLint: Uses `typescript-eslint` with recommended configs
- Prettier: Single quotes, trailing commas
- Run `npm run lint` before committing backend code
- Run `npm run format` to auto-format

---

### PostgreSQL / TypeORM

- Database is PostgreSQL; connection string comes from `DATABASE_URL`
- Entities live in per-module `persistence/` folders (and `persistence/entities/` for join tables)
- Use TypeORM decorators: `@Entity`, `@Column`, `@OneToMany` / `@ManyToOne`, `@PrimaryGeneratedColumn`
- Inject repositories with `@InjectRepository(Entity)` and `@nestjs/typeorm` `TypeOrmModule.forFeature([...])`
- `synchronize` is enabled in non-prod for convenience; **off** in prod (use `npm run db:migrate`)
- Localized strings (`title`, `slug`, `description`, `trailer`) are JSONB columns shaped as `{ "en-US"?: string, "nl-NL"?: string }`
- Polymorphic "media" references use `media_id` + `media_type` (`'video' | 'collection'`) with no DB FK; resolve in app code
- The `users` table is owned by **better-auth** (`npx auth migrate`); our `User` entity maps to it for reads/relations (custom `role` and `avatarB64` columns)

---

### Authentication

- Uses **better-auth** wired via `@thallesp/nestjs-better-auth` (`AuthModule.forRoot({ auth })` in `app.module.ts`)
- Auth endpoints are mounted under `/api/auth/*`; the better-auth instance lives in `src/auth/auth.ts`
- A global `AuthGuard` protects every route by default — use `@AllowAnonymous()` / `@OptionalAuth()` to open routes
- Inject the session with `@Session() session: UserSession` from `@thallesp/nestjs-better-auth`
- Roles: custom `role` column ('user' | 'admin') on better-auth's `users` table (`additionalFields`)
- `RolesGuard` + `@Roles(...roles)` (in `auth/middleware/`) read `session.user.role`; stack with `@UseGuards(RolesGuard)`
- `main.ts` bootstraps with `bodyParser: false` (required by the better-auth integration)
- Username login is enabled via the better-auth **username** plugin

---

### Zod Validation

- Validation schemas in `validation/` folders per module
- Use `createZodValidationPipe(Schema)` from `lib/utils/zod-validation.ts` on `@Body(...)`
- There is no global `ValidationPipe`; every validated route declares its own Zod pipe
- Shared schemas: `LocalizedStringSchema`, `PaginationSchema` in `lib/validation/`

---

### Key Dependencies

**Backend:**
- NestJS 11
- TypeORM 0.3 (PostgreSQL via `pg`)
- better-auth 1.6 (`@thallesp/nestjs-better-auth`)
- Zod 4
- fluent-ffmpeg, fast-glob, slugify, mime-types

**Frontend:**
- Nuxt 4
- Vue 3.5
- Tailwind CSS 4
- shadcn-vue (Reka UI)
- Zod 4

---

## 3. File Organization

```
backend/
├── src/
│   ├── auth/           # better-auth instance + RolesGuard / @Roles decorator
│   ├── users/          # User management (incl. anonymous register via invite code)
│   ├── videos/         # Video management (GET /api/videos/:id)
│   ├── collections/    # Collection management (+ genres, feeds: continue-watching/trending/top-rated)
│   ├── reviews/        # Reviews (create for all users, summary endpoint)
│   ├── watchlist/      # Per-user watchlist
│   ├── favorites/      # Per-user favorites
│   ├── invites/        # Admin-managed invite codes for registration
│   ├── images/         # Image handling
│   ├── stream/         # Video streaming (watch) + static files (images, subtitles)
│   ├── watch-progress/ # Progress tracking
│   ├── lib/            # Utilities (utils, validation)
│   └── main.ts

frontend/
├── app/
│   ├── components/     # Vue components
│   ├── pages/         # Route pages
│   ├── layouts/       # Layouts
│   ├── composables/   # (via Nuxt auto-import)
│   └── app.vue
├── nuxt.config.ts
└── package.json
```

---

## 4. Common Patterns

### Backend Service Pattern
```typescript
@Injectable()
export class VideoService {
  private readonly logger = new Logger(VideoService.name);

  constructor(
    @InjectRepository(Video) private videoRepository: Repository<Video>,
    private imageService: ImageService,
  ) {}

  async findVideoById(id: string): Promise<Video | null> {
    return this.videoRepository.findOne({
      where: { id },
      relations: { images: { image: true }, subtitles: true, persons: true },
    });
  }
}
```

### Frontend Component Pattern
```vue
<script setup>
const router = useRouter()

async function fetchData() {
  try {
    const { data } = await useFetch('/api/endpoint')
    return data
  } catch (err) {
    console.error(err)
  }
}
</script>

<template>
  <div>Component content</div>
</template>
```

---

## 5. Environment Variables

- Backend: `.env` file (see `.env.example`)
- Frontend: Nuxt config or `.env` files
- PostgreSQL connection string (`DATABASE_URL`) required for backend
- `BETTER_AUTH_SECRET` (≥ 32 chars) and `BETTER_AUTH_URL` required for authentication
- `FRONTEND_URL` configures CORS origin and better-auth trusted origins (defaults to `http://localhost:3000`)

---
> Source: [Leafar2001/dellekeshub-nuxt-nest](https://github.com/Leafar2001/dellekeshub-nuxt-nest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
