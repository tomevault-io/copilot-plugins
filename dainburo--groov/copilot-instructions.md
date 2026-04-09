## groov

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Groov is a full-stack dance move learning and choreography platform designed to help dancers remember and learn new dance moves, specifically focused on Lindy Hop style. The system allows users to browse dance moves, create sequences, and rate choreographies.

**Technology Stack:**
- **Frontend**: React 18 with TypeScript, React Router v6, SCSS modules, Axios for API calls
- **Backend**: Node.js with Express.js and InversifyJS for dependency injection
- **Database**: PostgreSQL with Knex.js as query builder and ORM
- **Authentication**: JWT-based auth with refresh tokens stored in HTTP-only cookies
- **API Documentation**: Swagger/OpenAPI available at `/api-docs`
- **Containerization**: Docker with Docker Compose for orchestration

## Development Commands

### Backend Commands (in `/backend` directory)
```bash
cd backend
npm run build          # Compile TypeScript to dist/
npm run dev            # Run development server with nodemon (hot reload)
npm start              # Run production build from dist/index.js
```

### Frontend Commands (in `/frontend` directory)
```bash
cd frontend
npm start              # Start React development server (port 3000)
npm run build          # Build production bundle
npm test               # Run tests
```

### Database Operations (in `/backend` directory)
```bash
cd backend
npm run migrate        # Run pending migrations and regenerate TypeScript types
npm run migration:add  # Create new migration file
npm run rollback       # Rollback last migration batch
npm run seed           # Run database seeding
```

The `migrate` command automatically regenerates the `backend/src/schema.ts` file with TypeScript types matching your database schema.

### Docker Operations (from project root)
```bash
docker-compose up           # Start all services (db, backend, frontend)
docker-compose up --build   # Rebuild and start all services
docker-compose down         # Stop all services
docker-compose logs -f      # View logs from all services
```

**Services and Ports:**
- **Database**: PostgreSQL on port 5435 (mapped from container 5432)
- **Backend**: Node.js/Express API on port 3003
- **Frontend**: React app served via nginx on port 3000

### Azure Deployment
For detailed Azure deployment instructions, see `AZURE_DEPLOYMENT.md`.

## Architecture

### Dependency Injection Pattern

This project uses **InversifyJS** for dependency injection with a three-layer architecture:

1. **Controllers** (`src/controllers/`) - HTTP request handlers decorated with `@controller` and `@httpPost`/`@httpGet`
2. **Services** (`src/services/`) - Business logic layer
3. **Repositories** (`src/repositories/`) - Data access layer extending `BaseRepository`

All dependencies are registered in `src/ioc/inversify.config.ts` using symbols defined in `src/ioc/types.ts`.

**Pattern Example:**
```typescript
// In controller
@controller('/auth')
export class AuthController extends BaseHttpController {
  constructor(@inject(TYPES.authService) private authService: AuthService) {
    super();
  }
}

// In service
@injectable()
export class AuthService {
  constructor(@inject(TYPES.authRepository) private authRepository: AuthRepository) {}
}
```

### Authentication System

- Custom `AuthProvider` implements `inversify-express-utils` authentication interface
- JWT access tokens and refresh tokens stored in HTTP-only cookies
- Token validation happens automatically via the auth provider on protected routes
- Use `@principal()` decorator in controllers to access authenticated user
- Access tokens expire, refresh tokens stored in database for rotation

### Database Layer

**BaseRepository Pattern:**
All repositories extend `BaseRepository` which provides:
- `get<T>()` - Type-safe SELECT queries
- `insert<T>()` - INSERT with returning
- `upsert<T>()` - INSERT with ON CONFLICT merge
- `delete()` - Soft delete (sets `deleted = true`)
- `hardDelete()` - Permanent deletion
- `usingTransaction()` - Transaction wrapper with auto-commit/rollback
- `withTransaction()` - Helper for using optional transaction context

**Auto-generated Types:**
Run `npm run migrate` after schema changes to regenerate TypeScript types in `src/schema.ts`. These include:
- Table enum with all table names
- TypeScript interfaces for each table
- Enums for custom database types (DifficultyEnum, KeyPositionEnum, RoleType)

### Domain Model

**Core Entities:**
- `app_user` - Users with role-based access (admin/user)
- `dance_move` - Individual dance moves with difficulty, start/end positions, and optional parent move
- `dance_sequence` - User-created sequences of moves, optionally tied to events
- `move_of_sequence` - Junction table linking moves to sequences with ordering
- `event` - Events that sequences can be associated with
- `rating` - User ratings for dance sequences
- `refresh_token` - Persistent refresh tokens for authentication

**Key Position System:**
Moves have `start_position` and `end_position` (both `KeyPositionEnum`) which enables the system to recommend compatible moves based on position matching.

## Important Patterns and Conventions

### Controller Registration
Controllers must be imported in `src/ioc/controllers.ts` for auto-discovery by InversifyJS:
```typescript
import '../controllers/yourController';
```

### Adding New Features
When adding a new entity:
1. Create migration in `migrations/` folder
2. Run `npm run migrate` to apply and regenerate types
3. Create repository extending `BaseRepository` in `src/repositories/`
4. Create service class decorated with `@injectable()` in `src/services/`
5. Create controller extending `BaseHttpController` in `src/controllers/`
6. Register all in `src/ioc/inversify.config.ts` and add symbols to `src/ioc/types.ts`
7. Import controller in `src/ioc/controllers.ts`

### Environment Configuration
Database connection settings are in `knexfile.ts`. For local development:
- Host: `127.0.0.1`
- Port: `5435`
- User: `admin`
- Password: `admin`
- Database: `groov_db`

For Docker, the backend service uses `DATABASE_URL` environment variable pointing to the `db` service.

### API Documentation
Swagger documentation is defined in `src/swagger/swagger.json` and served at `/api-docs` with persistent authorization enabled.

## Project Structure

```
Groov/
├── backend/              # Backend Node.js API
│   ├── src/
│   │   ├── authProvider/    # Custom auth provider
│   │   ├── controllers/     # HTTP endpoints
│   │   ├── errors/          # Custom error classes
│   │   ├── ioc/             # Dependency injection config
│   │   ├── jobs/            # Background tasks and seeds
│   │   ├── repositories/    # Data access layer
│   │   ├── services/        # Business logic layer
│   │   ├── swagger/         # API documentation
│   │   ├── types/           # TypeScript types
│   │   ├── schema.ts        # Auto-generated database types
│   │   └── index.ts         # Application entry point
│   ├── migrations/          # Database migration files
│   ├── Dockerfile          # Backend Docker configuration
│   ├── package.json
│   └── tsconfig.json
│
├── frontend/            # React TypeScript frontend
│   ├── public/          # Static assets
│   ├── src/
│   │   ├── components/      # React components
│   │   │   ├── common/      # Reusable UI components
│   │   │   └── layout/      # Layout components
│   │   ├── pages/           # Route-level page components
│   │   │   ├── Auth/        # Login, Signup
│   │   │   ├── DanceMoves/  # Dance move pages
│   │   │   ├── Sequences/   # Sequence pages
│   │   │   ├── Events/      # Event pages
│   │   │   └── Home/        # Landing page
│   │   ├── services/        # API service layer
│   │   ├── context/         # React Context (AuthContext)
│   │   ├── hooks/           # Custom React hooks
│   │   ├── types/           # TypeScript interfaces
│   │   ├── styles/          # SCSS design system
│   │   ├── utils/           # Helper functions
│   │   └── App.tsx          # Main app with routing
│   ├── Dockerfile          # Frontend Docker configuration
│   ├── nginx.conf          # Nginx configuration for production
│   ├── package.json
│   └── tsconfig.json
│
├── docker-compose.yml  # Orchestration for all services
├── CLAUDE.md           # This file
└── AZURE_DEPLOYMENT.md # Azure deployment guide
```

### Frontend Architecture

**React Application with TypeScript:**
- **Routing**: React Router v6 for client-side routing with protected routes
- **State Management**: React Context API for authentication state
- **API Layer**: Axios with interceptors for automatic token refresh
- **Styling**: SCSS modules with design system (variables, mixins, utilities)
- **Authentication**: Cookie-based JWT tokens, automatic refresh on 401 responses
- **Components**: Functional components with hooks, separated into common/layout/pages

**Key Frontend Features:**
- Protected routes requiring authentication
- Role-based UI rendering (admin vs user)
- Responsive design with mobile-first approach
- Comprehensive SCSS design system with variables and mixins
- Type-safe API calls with TypeScript interfaces matching backend models

**API Service Pattern:**
All API calls go through service modules (`authService`, `danceMoveService`, etc.) that use a centralized Axios instance with:
- Automatic cookie handling (`withCredentials: true`)
- Token refresh interceptor for seamless re-authentication
- Type-safe request/response handling

## Environment Variables

### Backend Environment Variables (in `/backend/.env.example`)

**Server Configuration:**
- `PORT` - Server port (default: 3003)
- `NODE_ENV` - Environment mode (development/production)

**Database Configuration:**
- `DB_HOST` - PostgreSQL host
- `DB_PORT` - PostgreSQL port (5435 for local, 5432 for Azure)
- `DB_USER` - Database username
- `DB_PASSWORD` - Database password
- `DB_NAME` - Database name
- `DB_SSL` - Enable SSL (true for Azure, false for local)

**Authentication:**
- `ACCESS_TOKEN_SECRET` - JWT access token secret
- `REFRESH_TOKEN_SECRET` - JWT refresh token secret

### Frontend Environment Variables (in `/frontend/.env.example`)
- `REACT_APP_API_URL` - Backend API URL (default: http://localhost:3003)

## Deployment

### Local Development with Docker
```bash
docker-compose up  # Starts PostgreSQL and runs migrations automatically
```

### Production Deployment to Azure
1. Review `AZURE_DEPLOYMENT.md` for complete deployment instructions
2. Configure environment variables in Azure App Service or Container Instance
3. Ensure PostgreSQL firewall rules allow Azure services
4. The `docker-entrypoint.sh` script automatically runs migrations on startup
5. Database seeding is skipped in production (set `RUN_SEED=true` to enable)

## Notes

- The project uses `strict: false` in TypeScript config due to decorator metadata
- All services and repositories use singleton scope
- Soft deletes are the default (use `hardDelete()` for permanent removal)
- CORS is configured for `http://localhost:3000` (React frontend)
- Request/response logging middleware is enabled for debugging
- The `knexfile.ts` now reads from environment variables for both development and production
- Docker builds use multi-stage builds to minimize final image size
- Frontend uses cookie-based authentication (no localStorage for tokens)
- Nginx proxy forwards `/api/*` requests to backend in production

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DainBuro)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/DainBuro)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
