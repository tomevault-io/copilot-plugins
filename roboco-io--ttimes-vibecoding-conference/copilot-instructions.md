## ttimes-vibecoding-conference

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VibeCoding Todo App is a live demo project for learning vibe coding techniques. It's a full-stack Todo management application with Go backend and React frontend, designed to be implemented rapidly using MVP approach.

## Architecture

### High-Level Structure
- **Backend**: Go + Gin/Gorilla Mux + SQLite + JWT authentication
- **Frontend**: React 18 + Vite + Tailwind CSS + React Query + React Router
- **Infrastructure**: Docker + Docker Compose for local containerized deployment
- **Database**: SQLite with file-based storage for simplicity

### Planned Directory Structure
```
backend/          # Go API server
frontend/         # React application  
docker-compose.yml # Container orchestration
docs/             # Project documentation
```

### API Design
Authentication endpoints:
- `POST /api/auth/register` - User registration
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout

Todo CRUD endpoints:
- `GET /api/todos` - List user's todos
- `POST /api/todos` - Create new todo
- `PUT /api/todos/:id` - Update todo
- `DELETE /api/todos/:id` - Delete todo
- `PATCH /api/todos/:id` - Toggle todo status

### Database Schema
**Users table**: id, email, password_hash, created_at, updated_at
**Todos table**: id, user_id, title, description, completed, created_at, updated_at

## Development Commands

### Backend (when implemented)
```bash
# Run Go server
go run main.go

# Run tests
go test ./...

# Build binary
go build -o todo-api
```

### Frontend (when implemented)
```bash
# Install dependencies
npm install

# Start dev server
npm run dev

# Build for production
npm run build

# Run tests
npm test
```

### Docker Commands
```bash
# Build and run all services
docker-compose up --build

# Run in background
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs
```

## Development Phases

**Phase 1**: Backend API development (Go project setup, SQLite schema, auth + CRUD APIs)
**Phase 2**: Frontend UI development (React setup, Tailwind styling, auth + todo pages)  
**Phase 3**: Integration and deployment (API integration, Docker setup, local testing)
**Phase 4**: Live demo preparation (bug fixes, optimization)

## Key Constraints

- Demo/learning purpose - avoid complex enterprise features
- No email verification required for registration
- SQLite only - single server deployment
- Local container deployment only
- MVP approach - implement core features first

## Success Criteria

- User registration/login working
- Full Todo CRUD functionality
- Frontend-backend API integration complete
- Responsive UI implementation
- Docker containers running successfully
- API response times under 500ms

---
> Source: [roboco-io/ttimes-vibecoding-conference](https://github.com/roboco-io/ttimes-vibecoding-conference) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
