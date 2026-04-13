## noted

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Noted is a modern, simplistic board where anyone can anonymously post text notes. It consists of a Go REST API backend and an Angular frontend client.

## Architecture

**Backend (Go):**
- Clean architecture with layered structure: controller → service → repository → database
- PostgreSQL database with a simple `note` table (id, text)
- REST API endpoints at `/api/note/*` for CRUD operations
- CORS enabled for frontend communication on `http://localhost:4200`
- Server runs on `localhost:8888`

**Frontend (Angular 17):**
- Standard Angular CLI project structure
- Services for API communication (`api.service.ts`, `note.service.ts`)
- Home component for displaying notes
- TypeScript with strict configuration

**Data Flow:**
1. Angular frontend makes HTTP requests to Go backend
2. Go controllers handle requests and delegate to services
3. Services contain business logic and call repositories
4. Repositories interact with PostgreSQL database
5. Response flows back through the layers

## Development Commands

**Frontend (run from `/client` directory):**
```bash
npm run start    # Start dev server on localhost:4200
npm run build    # Build for production
npm run test     # Run unit tests with Karma
ng generate component component-name  # Generate new component
```

**Backend (run from `/server` directory):**
```bash
go run main.go   # Start server on localhost:8888
go mod tidy      # Clean up dependencies
```

**Database Setup:**
- Create PostgreSQL database named `noted_dev`
- Run `/server/sql/note_table.sql` to create the note table
- Default connection: `postgres:password@localhost:5432/noted_dev`
- Configure database connection in `/server/config/database.go`

## Key Directories

- `/client/src/app/` - Angular components and services
- `/client/src/types.ts` - TypeScript interfaces (Note, NotesResponse, Options)
- `/server/controller/` - HTTP request handlers
- `/server/service/` - Business logic layer
- `/server/repository/` - Database access layer
- `/server/model/` - Data structures
- `/server/data/` - Request/response DTOs
- `/server/helper/` - Utility functions (JSON, error handling)
- `/server/config/` - Database configuration

## API Endpoints

- `GET /api/note` - Get all notes
- `GET /api/note/:noteId` - Get note by ID
- `POST /api/note` - Create new note
- `PATCH /api/note/:noteId` - Update note
- `DELETE /api/note/:noteId` - Delete note

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ianwood103)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ianwood103)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
