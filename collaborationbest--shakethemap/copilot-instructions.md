## shakethemap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ShakeTheMap is a fullstack job matching platform with:
- **Backend**: NestJS (TypeScript) REST API with JWT authentication
- **Frontend**: Next.js 13 with App Router, React components, and TailwindCSS
- **Database**: MySQL 8.0 with TypeORM
- **Deployment**: Docker & Docker Compose

### Docker Development
```bash
# From project root
docker compose -f docker-compose/development.yml up --build 
# Frontend: http://localhost:3000
# Backend: http://localhost:5000
# MySQL: localhost:3307
```

## Architecture Overview

### Backend Structure
- **Modular Architecture**: Each feature is a separate module (auth, users, chat, alerts, profiles)
- **Authentication**: JWT-based with access/refresh tokens, Passport strategies
- **Database**: MySQL with TypeORM, auto-synchronize enabled in development
- **Key Modules**:
  - `auth/`: Authentication system with local and JWT strategies, password reset
  - `users/`: User management with role-based access (employee/recruiter sub-roles)
  - `chat/`: Real-time messaging system with conversations and messages
  - `alerts/`: Notification system
  - `profiles/`: Employee profiles with sector-specific entities (bakery, butchery, tertiary)
  - `profilesPrimary/`: Primary sector profiles (agriculture, construction, etc.)
  - `profilesTertiary/`: Tertiary sector profiles (services, office work, etc.)
  - `offers/`: Job offers with contract types and salary entities
  - `offersBakery/`: Specialized bakery job offers with industry-specific fields
  - `applications/`: Job applications with status tracking
  - `notifications/`: System notifications with different types
  - `marketing/`: Waiting list management for marketing campaigns
  - `brevo/`: Email service integration for transactional emails
  - `checklist/`: User onboarding and task completion tracking

### Frontend Structure
- **App Router**: Next.js 13+ with TypeScript
- **Route Groups**: `(auth)`, `(employee)`, `(recruiter)`, `(public)`
- **UI Components**: Radix UI + TailwindCSS with shadcn/ui
- **State Management**: React hooks with custom providers
- **Key Features**:
  - Authentication flows with protected routes
  - Interactive maps using Leaflet/React-Leaflet
  - Real-time messaging interface
  - Profile management systems
  - Job search and application workflows

### Database Schema
- **Users**: Core user entity with role-based access (employee/recruiter)
- **Profiles**: Separate employee and professional profile entities with skills and preferences
- **Chat**: Conversations and messages for real-time communication between users
- **Alerts**: User-specific notification system
- **Offers**: Job postings with contract types, salary ranges, and locations
- **Applications**: Job application tracking with status management
- **Notifications**: System-wide notifications with types and read status
- **Addresses**: Location data for offers and profiles
- **Marketing**: Waiting list for pre-launch marketing campaigns

## Development Environment

### Environment Variables
Required for backend:
- `DB_HOST`, `DB_USERNAME`, `DB_PASSWORD`, `DB_DATABASE`
- `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`
- `JWT_ACCESS_EXPIRES_IN`, `JWT_REFRESH_EXPIRES_IN`

Required for frontend:
- `NEXT_PUBLIC_API_URL`: Backend API URL
- `NEXT_PUBLIC_FRONTEND_URL`: Frontend URL

### Key Dependencies
- **Backend**: NestJS, TypeORM, MySQL2, Passport, Zod validation with @anatine/zod-nestjs, bcrypt, class-validator, axios
- **Frontend**: Next.js 13, React 18, TailwindCSS, Radix UI components, React Hook Form, Leaflet/React-Leaflet, shadcn/ui, Lucide icons

## Code Conventions

### Backend
- TypeScript strict mode enabled
- Zod schemas for validation with `@anatine/zod-nestjs`
- Entity-first database design with TypeORM decorators
- Module-based architecture with dependency injection
- Guards for route protection (JWT, Local)

### Frontend
- TypeScript with strict configuration
- Component-based architecture with shadcn/ui patterns
- TailwindCSS for styling with custom design system
- React Hook Form with Zod validation
- Custom hooks for API integration and state management

## Testing & Development Scripts

### Backend Scripts
- `npm run build`: Build the application
- `npm run format`: Format code with Prettier
- `npm run start`: Start the application
- `npm run start:dev`: Start in development mode with watch
- `npm run start:debug`: Start in debug mode with watch
- `npm run lint`: Run ESLint with auto-fix
- `npm run test`: Run Jest unit tests
- `npm run test:watch`: Run tests in watch mode
- `npm run test:cov`: Run tests with coverage
- `npm run test:debug`: Run tests in debug mode
- `npm run test:e2e`: Run end-to-end tests with Supertest
- `npm run seed`: Seed database with test data
- `npm run seed:users`: Seed only user data

### Frontend Scripts
- `npm run dev`: Start development server
- `npm run build`: Build for production (with disabled Google Fonts optimization)
- `npm run start`: Start production server
- `npm run lint`: Run Next.js ESLint

### Testing Strategy
- Backend: Jest for unit tests, Supertest for e2e tests
- Frontend: Next.js built-in ESLint configuration
- Test files follow `*.spec.ts` and `*.e2e-spec.ts` patterns


## Docker Development Commands
- The project containers are typically already running in development
- To run commands in backend container: `docker exec docker-compose-shakethemap-backend-1 [command]`
- To run commands in frontend container: `docker exec docker-compose-shakethemap-frontend-1 [command]`
- To install packages: `docker exec docker-compose-shakethemap-backend-1 npm install [package]`
- Always edit files directly in the local filesystem, not via docker exec
- Container names follow pattern: `docker-compose-shakethemap-[service]-1`

## Important Notes
- Database synchronization is not enabled in development (`synchronize: false`)
- ESLint errors are ignored during builds in frontend
- Images are unoptimized in Next.js config for development
- MySQL runs on custom port 3307 in development to avoid conflicts
- JWT tokens are handled via HTTP-only cookies for security
- Google Fonts CSS optimization is disabled in frontend build
- Frontend uses custom fonts (Barlow, Changa One) from @fontsource
- Backend includes comprehensive seeding system for development data
- Brevo integration for transactional email services
- Application status tracking with enum-based states
- Real-time messaging system with conversation management



## Backend Development Rules
- **Response DTOs**: Controllers must always return types extending `BaseResponseDto<T>`
  - Include `correlationId` in meta field for request tracking
  - Use consistent error structure with code, message, and optional details
- **DTOs**: Every controller method and service function requires input and output DTOs
- **Correlation ID**: Controllers and services must use and log the `correlationId` for tracing
- **Validation**: Use Zod schemas with `@anatine/zod-nestjs` for request validation
- **Architecture**: Follow modular structure with clear separation of concerns

## Frontend Development Rules
- **Button Components**: Always use `ui/buttons/Button` instead of creating new button components
- **Color System**: Use CSS custom properties from `globals.css`, never hardcode colors
  - Available job type colors: `--job-1-primary` through `--job-7-primary` with corresponding secondary variants
  - Typography colors: Use predefined classes like `.titre`, `.sous-titre`, `.texte-normal`, etc.
  - Add new colors to the `:root` section in `globals.css`
- **Typography**: Use predefined CSS classes from `globals.css`:
  - `.titre`: Main headings (Changa One, 56px)
  - `.sous-titre`: Section headings (Changa One, 32px)
  - `.texte-normal`: Body text (Barlow, 16px)
  - `.texte-important`: Important text (Barlow Bold, 16px)
- **Job Sector Colors**: Each job sector has dedicated color schemes:
  - Job 1 (Alimentation/Restauration): `--job-1-primary` (#FA3F42), `--job-1-secondary` (#FFDADB)
  - Job 2 (Santé/Soins): `--job-2-primary` (#519EF6), `--job-2-secondary` (#E0EFFF)
  - Job 3 (Mode/Beauté): `--job-3-primary` (#942761), `--job-3-secondary` (#FFD5EB)
  - Job 4 (Bâtiment/Construction): `--job-4-primary` (#414141), `--job-4-secondary` (#C7C7C7)
  - Job 5 (Mécanique/Automobile): `--job-5-primary` (#EF773F), `--job-5-secondary` (#FFE9DF)
  - Job 6 (Multimédia/Spectacle): `--job-6-primary` (#3136B4), `--job-6-secondary` (#C5C7EF)
  - Job 7 (Espaces verts/Nature): `--job-7-primary` (#279474), `--job-7-secondary` (#D1FBEF)

---
> Source: [collaborationbest/shakethemap](https://github.com/collaborationbest/shakethemap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
