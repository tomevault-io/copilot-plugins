## project-with-gcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack Todo application built with TypeScript, consisting of three main components:
- **Frontend**: Next.js 14 with React 18, TypeScript, and Tailwind CSS
- **Backend**: Express.js REST API with TypeScript and Node.js 20
- **Database**: MySQL 8.0

The application is fully containerized using Docker and Docker Compose, and can be deployed to Google Cloud Platform (GCP) using Terraform.

## Development Commands

### Docker Environment

Start all services (database, backend, frontend):
```bash
docker-compose up
```

Start services in detached mode:
```bash
docker-compose up -d
```

Stop all services:
```bash
docker-compose down
```

Stop and remove volumes (clears database):
```bash
docker-compose down -v
```

Rebuild containers after dependency changes:
```bash
docker-compose up --build
```

### Backend (Express API with TypeScript)

Location: `backend/`

Development (with hot reload using ts-node-dev):
```bash
cd backend
npm run dev
```

Build TypeScript:
```bash
cd backend
npm run build
```

Production (requires build first):
```bash
cd backend
npm run build
npm start
```

Install dependencies:
```bash
cd backend
npm install
```

The backend runs on port 8080 by default.

### Frontend (Next.js)

Location: `frontend/`

Development server:
```bash
cd frontend
npm run dev
```

Production build:
```bash
cd frontend
npm run build
npm start
```

Linting:
```bash
cd frontend
npm run lint
```

The frontend runs on port 3000 by default.

## Architecture

### Backend API Structure

The backend is a TypeScript Express application with organized modules:

**Project Structure**:
- `src/server.ts` - Main Express server and application entry point
- `src/routes/todos.ts` - Todo API route handlers
- `src/config/database.ts` - MySQL connection pool and database initialization
- `src/types/todo.ts` - TypeScript type definitions for Todo entities

**Database Initialization**: On startup (`initDatabase()` in `src/config/database.ts`), the server automatically creates the `todos` table if it doesn't exist using a connection pool pattern.

**API Endpoints**:
- `GET /health` - Health check endpoint
- `GET /api/todos` - Fetch all todos
- `GET /api/todos/:id` - Fetch a specific todo
- `POST /api/todos` - Create a new todo (requires `title`, optional `description`)
- `PUT /api/todos/:id` - Update a todo (supports partial updates: `title`, `description`, `completed`)
- `DELETE /api/todos/:id` - Delete a specific todo
- `DELETE /api/todos` - Delete all todos

**Database Schema**:
```sql
todos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  completed BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
)
```

**Connection Pool**: Uses `mysql2/promise` with a connection pool (limit: 10 connections) for efficient database access. TypeScript types from mysql2 (`ResultSetHeader`, `RowDataPacket`) are used for type-safe database queries.

**TypeScript Types**: The application uses strict TypeScript typing:
- `Todo` - Full todo entity with all fields
- `CreateTodoInput` - Input type for creating todos
- `UpdateTodoInput` - Input type for updating todos (all fields optional)

### Frontend Structure

The frontend is a Next.js 14 App Router application with TypeScript and client-side rendering:

**Project Structure**:
- `src/app/page.tsx` - Main todo page (client component)
- `src/app/layout.tsx` - Root layout with metadata
- `src/app/globals.css` - Global styles with Tailwind directives

**Main Page** (`frontend/src/app/page.tsx`): A single-page client component with TypeScript interfaces that handles all todo operations using React hooks and fetch API.

**State Management**: Uses React useState for local state:
- Todo list
- Form inputs (title, description)
- Loading and error states

**API Integration**: Communicates with the backend via `NEXT_PUBLIC_API_URL` environment variable (defaults to `http://localhost:8080`).

**User Actions**:
- Add todos via form submission
- Toggle completion status by clicking todos
- Delete todos with confirmation dialog
- Auto-refresh after mutations

### Docker Architecture

**Networks**: All services communicate over a bridge network (`todo-network`).

**Service Dependencies**:
1. MySQL starts first with health checks
2. Backend waits for MySQL to be healthy
3. Frontend depends on backend

**Volume Mounts**:
- MySQL data persisted in `mysql_data` volume
- Development: Source code mounted with node_modules excluded for hot reload

**Environment Variables**:
- Backend: `PORT`, `NODE_ENV`, `DB_HOST`, `DB_USER`, `DB_PASSWORD`, `DB_NAME`
- Frontend: `NEXT_PUBLIC_API_URL`
- MySQL: `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, `MYSQL_PASSWORD`

### Database Credentials

Default credentials (configured in `docker-compose.yml`):
- Root password: `rootpassword`
- Application user: `todoapp_user`
- Application password: `todoapp_pass`
- Database name: `todo_db`

For local development, these should be overridden in `backend/.env` file (see `backend/.env.example`).

## Port Mapping

- Frontend: http://localhost:3000
- Backend API: http://localhost:8080
- MySQL: localhost:3306

## Key Patterns

### Error Handling

- Backend returns appropriate HTTP status codes (404, 400, 500) with JSON error messages
- Frontend displays error messages to users and logs to console
- MySQL connection errors are caught during initialization

### Data Flow

1. User interacts with Next.js frontend
2. Frontend makes fetch requests to Express API
3. Express queries MySQL using connection pool
4. Response flows back through the stack
5. Frontend updates UI with new data

### Development Workflow

When modifying the backend:
1. Changes auto-reload via `ts-node-dev` in Docker (watches TypeScript files)
2. TypeScript compilation happens automatically in development mode
3. For production, run `npm run build` to compile TypeScript to JavaScript in the `dist/` folder
4. Database schema changes require manual migration or container restart with volume deletion

When modifying the frontend:
1. Next.js hot reloads automatically with TypeScript support
2. Environment variables must be prefixed with `NEXT_PUBLIC_` to be accessible in browser
3. TypeScript errors are shown in the terminal and browser during development

### TypeScript Configuration

**Backend** (`backend/tsconfig.json`):
- Target: ES2020
- Module: CommonJS
- Strict mode enabled
- Output directory: `dist/`
- Source directory: `src/`

**Frontend** (`frontend/tsconfig.json`):
- Next.js App Router configuration
- Strict mode enabled
- Path aliases: `@/*` maps to `./src/*`
- JSX: preserve (handled by Next.js)

## GCP Deployment

This project can be deployed to Google Cloud Platform using Terraform and Cloud Run.

### Infrastructure Components

**Cloud Services**:
- **Cloud Run**: Serverless container platform for backend and frontend
- **Cloud SQL**: Managed MySQL 8.0 database
- **VPC Connector**: Private connection between Cloud Run and Cloud SQL
- **Artifact Registry**: Docker image storage
- **Secret Manager**: Secure storage for database credentials
- **Cloud Build**: CI/CD pipeline (optional)

**Terraform Configuration** (`terraform/`):
- `main.tf` - Provider and API enablement
- `network.tf` - VPC, subnet, and VPC connector
- `cloudsql.tf` - Cloud SQL instance and database
- `cloudrun.tf` - Backend and frontend Cloud Run services
- `secrets.tf` - Secret Manager configuration
- `artifact_registry.tf` - Docker repository
- `variables.tf` - Input variables
- `outputs.tf` - Output values (URLs, database info)

### Deployment Process

1. **Initial Setup**:
   ```bash
   gcloud config set project YOUR_PROJECT_ID
   ./scripts/setup-gcp.sh
   ```

2. **Configure Terraform**:
   ```bash
   cd terraform
   cp terraform.tfvars.example terraform.tfvars
   # Edit terraform.tfvars with your project details
   ```

3. **Deploy**:
   ```bash
   ./scripts/deploy.sh
   ```

4. **Access Application**:
   ```bash
   terraform -chdir=terraform output frontend_url
   ```

### Production Docker Images

Separate production Dockerfiles are provided:
- `backend/Dockerfile.prod` - Multi-stage build with TypeScript compilation
- `frontend/Dockerfile.prod` - Optimized Next.js standalone build

Key differences from development:
- TypeScript compiled to JavaScript
- Dependencies pruned to production only
- Non-root user for security
- Optimized layer caching

### CI/CD with Cloud Build

`cloudbuild.yaml` provides automated deployment:
1. Build backend and frontend Docker images
2. Push images to Artifact Registry
3. Deploy to Cloud Run

Connect to GitHub for automatic deployment on push.

### Cost Estimation

Approximate monthly costs (light usage):
- Cloud Run: $5-10
- Cloud SQL (db-f1-micro): $7-15
- VPC Connector: $7
- Total: ~$20-32/month

See `GCP_DEPLOYMENT.md` for detailed deployment guide and troubleshooting.

### Monitoring and Management

**View Logs**:
```bash
gcloud run logs read todo-backend --region=asia-northeast1
gcloud run logs read todo-frontend --region=asia-northeast1
```

**Database Connection**:
```bash
cloud-sql-proxy YOUR_PROJECT_ID:asia-northeast1:todo-app-mysql
```

**Cleanup**:
```bash
cd terraform && terraform destroy
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NishimuraShiro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
