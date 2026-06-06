## claude-template

> This is a production-ready backend HTTP service template designed for use with Claude. It provides a complete foundation for building Express.js applications with TypeScript, PostgreSQL, Docker, and comprehensive testing.

# Claude Template - Backend HTTP Service

This is a production-ready backend HTTP service template designed for use with Claude. It provides a complete foundation for building Express.js applications with TypeScript, PostgreSQL, Docker, and comprehensive testing.

**🌐 Live Demo:** https://claude-template-api.onrender.com
- **Health Check**: https://claude-template-api.onrender.com/health
- **API Docs**: https://claude-template-api.onrender.com/api-docs
- **Try it**: `curl https://claude-template-api.onrender.com/todo/list`

## 🚀 Quick Start

### Local Development
**Prerequisites:** Docker and Docker Compose

```bash
# 1. Start all services
npm run docker:up

# 2. Run database migrations
npm run docker:migrate

# 3. Seed with test data (optional)
npm run docker:seed

# 4. Test the API
curl http://localhost:3000/health
curl http://localhost:3000/todo/list
```

**Local Access Points:**
- **API Server**: http://localhost:3000
- **Swagger Docs**: http://localhost:3000/api-docs
- **pgAdmin**: http://localhost:5050 (admin@example.com / admin123)
- **Database**: PostgreSQL on port 5432

### Production Deployment (Render)
This template includes complete Render.com deployment configuration:

```bash
# 1. Push to GitHub
git push origin main

# 2. Connect to Render.com
# - Create new Blueprint from your repo
# - Render auto-detects render.yaml configuration
# - Creates PostgreSQL database + web service

# 3. Your API goes live automatically!
```

## Architecture Overview

This template follows an action-based API design pattern rather than traditional RESTful conventions. Key architectural decisions:

- **Action-based endpoints**: Uses explicit action names (e.g., `/todo/create`, `/todo/remove`) instead of REST verbs
- **Interactors pattern**: Business logic is organized in interactors, not controllers or services
- **Serializers**: Dedicated layer for response formatting and data transformation
- **TypeScript-first**: Full TypeScript support with strict mode enabled
- **Docker-first**: Complete containerized development environment
- **Test-driven**: Co-located test files with Jest configured

## Project Structure

```
src/
  routes/         # Express route definitions (action-based endpoints)
  interactors/    # Business logic layer (all domain logic lives here)
  serializers/    # Response formatting and data transformation
  models/         # Sequelize models for PostgreSQL
  middleware/     # Express middleware (error handling, validation, auth)
  config/         # Configuration files (database, swagger, app settings)
  utils/          # Utility functions and helpers
  migrations/     # Database migrations
  seeders/        # Database seed files
```

## Key Technologies

- **Runtime**: Node.js (latest version) with CommonJS
- **Framework**: Express.js with TypeScript
- **Database**: PostgreSQL with Sequelize ORM
- **Containerization**: Docker & Docker Compose
- **Testing**: Jest with co-located test files
- **Documentation**: Swagger/OpenAPI with decorators
- **Development**: tsx watch for hot reloading
- **Code Quality**: ESLint, Prettier, TypeScript strict mode

## API Design Philosophy

This template uses **action-based endpoints** instead of RESTful design:

```typescript
// Instead of RESTful:
// PUT /todos/:id
// DELETE /todos/:id

// We use action-based:
// POST /todo/create
// POST /todo/remove
// POST /todo/complete
// POST /todo/assign
```

This makes APIs more explicit and easier to understand.

## Docker Commands (Recommended)

```bash
# Service Management
npm run docker:up        # Start all services (PostgreSQL + app)
npm run docker:down      # Stop all services
npm run docker:logs      # View application logs
npm run docker:build     # Rebuild Docker images

# Database Operations  
npm run docker:migrate   # Run database migrations
npm run docker:seed      # Seed database with test data

# Development (if not using Docker)
npm run dev             # Start development server with hot reload
npm run build           # Build TypeScript to JavaScript
npm run test            # Run Jest tests
npm run test:watch      # Run tests in watch mode
npm run lint            # Run ESLint
npm run typecheck       # Run TypeScript type checking
```

## Testing Strategy

- Tests are co-located with source files (e.g., `create-todo.interactor.test.ts`)
- Unit tests for interactors focus on business logic
- Integration tests for routes test the full request/response cycle
- Use `supertest` for API endpoint testing
- Database is mocked or uses a test database

## Environment Variables

Create a `.env` file based on `.env.example`:

```env
NODE_ENV=development
PORT=3000
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
JWT_SECRET=your-secret-key
```

## Adding New Features

1. **Create the model** in `src/models/`
2. **Create interactors** in `src/interactors/[domain]/` with tests
3. **Create serializers** in `src/serializers/`
4. **Define routes** in `src/routes/` with action-based endpoints
5. **Add Swagger documentation** using decorators
6. **Create migrations** if database schema changes

## Code Conventions

- Use async/await instead of callbacks
- Throw errors in interactors, handle them in middleware
- Keep interactors pure and testable
- Use TypeScript types/interfaces, avoid `any`
- Follow action-based naming: `createTodo`, `removeTodo`, etc.

## Common Patterns

### Interactor Example
```typescript
// src/interactors/todo/create-todo.interactor.ts
export async function createTodo(data: CreateTodoInput): Promise<Todo> {
  // Business logic here
  return await Todo.create(data);
}
```

### Route Example
```typescript
// src/routes/todo.routes.ts
router.post('/todo/create', async (req, res, next) => {
  try {
    const todo = await createTodo(req.body);
    res.json(serializeTodo(todo));
  } catch (error) {
    next(error);
  }
});
```

## Working Example Endpoints

The template includes a complete Todo domain with these working endpoints:

### Local Development
```bash
# Health Check
curl http://localhost:3000/health

# List todos (with pagination, filtering)
curl http://localhost:3000/todo/list
curl "http://localhost:3000/todo/list?completed=false&priority=high&limit=5"

# Create todo
curl -X POST http://localhost:3000/todo/create \
  -H "Content-Type: application/json" \
  -d '{"title": "New task", "description": "Task details", "priority": "high"}'

# Complete todo
curl -X POST http://localhost:3000/todo/complete \
  -H "Content-Type: application/json" \
  -d '{"id": 1}'

# Remove todo
curl -X POST http://localhost:3000/todo/remove \
  -H "Content-Type: application/json" \
  -d '{"id": 1}'
```

### Live Production API
```bash
# Health Check
curl https://claude-template-api.onrender.com/health

# List todos
curl https://claude-template-api.onrender.com/todo/list

# Create todo
curl -X POST https://claude-template-api.onrender.com/todo/create \
  -H "Content-Type: application/json" \
  -d '{"title": "Live todo", "priority": "high"}'

# Complete todo (replace ID with actual ID from create response)
curl -X POST https://claude-template-api.onrender.com/todo/complete \
  -H "Content-Type: application/json" \
  -d '{"id": 1}'
```

## Deployment Files

This template includes complete deployment configuration:

**Render.com Deployment:**
- `render.yaml` - Defines web service + PostgreSQL database
- `Dockerfile.render` - Production-optimized Docker build
- `.renderignore` - Excludes unnecessary files from deployment

**Development:**
- `docker-compose.dev.yml` - Local development with hot reloading
- `Dockerfile.dev` - Development Docker image
- `.sequelizerc` - Environment-aware Sequelize configuration

## Common Issues & Solutions

**Sequelize Model Fields:**
- Use `declare` keyword instead of `public` to avoid shadowing getters/setters
- Migration files must use `.cjs` extension for ES module compatibility

**Docker Development:**
- Database persists between restarts via Docker volumes
- Hot reloading works automatically with volume mounts
- Use `npm run docker:build` after package.json changes

**Render Deployment:**
- `sequelize-cli` must be in production dependencies for migrations
- Config files need to be preserved after build for Sequelize CLI
- Auto-deploys on GitHub push to main branch

## Template Achievements

✅ **Complete Backend Service Template:**
- Express.js + TypeScript with strict mode
- PostgreSQL database with Sequelize ORM
- Action-based API design (not RESTful)
- Comprehensive Jest testing with co-located tests
- Swagger/OpenAPI documentation
- Docker development environment
- Production deployment to Render.com

✅ **Live Production Example:**
- **API**: https://claude-template-api.onrender.com
- **Docs**: https://claude-template-api.onrender.com/api-docs
- Auto-deploys from GitHub pushes
- Managed PostgreSQL database
- Health monitoring and logging

## Important Notes for Claude

- **Docker-first development**: Always use Docker commands for consistency
- **Action-based endpoints**: Use `/todo/create`, not `POST /todos`
- **Interactors pattern**: Keep all business logic in interactors only
- **Co-located tests**: Write tests next to source files
- **Serializer pattern**: Use serializers for all API responses
- **TypeScript strict**: Follow strict mode requirements
- **Database files**: Migrations and seeds must use `.cjs` extension
- **Production ready**: Template includes complete Render deployment config

This template provides a **production-ready foundation** for any backend HTTP service project.

---
> Source: [wSnarski/claude-template](https://github.com/wSnarski/claude-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
