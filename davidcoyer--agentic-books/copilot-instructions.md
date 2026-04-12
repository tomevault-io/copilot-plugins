## agentic-books

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A minimal full-stack book library application built for an AI coding agents workshop. The stack consists of:
- **Frontend**: Vue 3 + Vite (port 5173)
- **Backend**: .NET 9.0 Minimal API with Entity Framework Core InMemory database (port 5298)

## Development Commands

### Frontend
```bash
npm install          # Install dependencies
npm run dev          # Start dev server (http://localhost:5173)
npm run build        # Build for production
npm run preview      # Preview production build
```

### Backend
```bash
cd backend
dotnet run           # Start API server (http://localhost:5298)
dotnet build         # Build the project

# Using Docker (no .NET installation required)
docker build -t agentic-books-backend .
docker run -p 5298:5298 agentic-books-backend
```

### Running the Full Application
Start both servers in separate terminals:
1. Terminal 1: `npm run dev` (frontend)
2. Terminal 2: `cd backend && dotnet run` (backend)
   - Or with Docker: `cd backend && docker build -t agentic-books-backend . && docker run -p 5298:5298 agentic-books-backend`

## Architecture

### Multi-User System
The application implements a simple multi-user system using the `X-User-Id` header:
- Frontend randomly selects a USER_ID (1-10) on page load in `src/services/bookService.js`
- All API requests include this USER_ID in the `X-User-Id` header
- Backend filters books by UserId to ensure users only see their own data
- The in-memory database is seeded with 5 books per user (50 books total)

### Frontend Structure
- **App.vue**: Root component, renders BookList
- **BookList.vue**: Main container managing book state, search/filter UI, and add book form
- **BookItem.vue**: Individual book card with inline edit mode
- **bookService.js**: API communication layer with USER_ID management

### Backend Structure
- **Program.cs**: Minimal API with CRUD endpoints, CORS configuration, and data seeding
- **Book.cs**: Entity model with Id, Title, Author, Genre, Year, Pages, and UserId
- **AppDbContext.cs**: EF Core DbContext for the in-memory database

### Key API Endpoints
All endpoints require `X-User-Id` header and automatically filter by user:
- `GET /books` - Fetch all books for current user
- `GET /books/{id}` - Fetch single book (user ownership verified)
- `POST /books` - Add new book (UserId assigned from header)
- `PUT /books/{id}` - Update book (user ownership verified)
- `DELETE /books/{id}` - Delete book (user ownership verified)

### State Management Pattern
The frontend uses local reactive state in BookList.vue with immediate UI updates:
1. User action updates local `books` ref array
2. Optimistically updates UI
3. Calls API function to sync with backend
4. API errors are not currently handled (workshop simplicity)

## Important Implementation Details

### API URL Configuration
The API URL is hardcoded in `src/services/bookService.js` as `http://localhost:5298/books`. When changing ports or deploying, update this constant.

### User Isolation
The UserId is critical for data isolation. When adding features that modify books:
- Always include `X-User-Id` header in API requests
- Backend validates user ownership before any modification
- Never expose or allow modification of the UserId field in the UI

### In-Memory Database
The backend uses EF Core InMemory database, meaning:
- All data is lost when the backend restarts
- Data is re-seeded on startup if the Books table is empty
- Not suitable for production use

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidcoyer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/davidcoyer)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
