## housing-webpage-login

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Architecture

This is a full-stack web application for housing data with user authentication:

- **Frontend**: React + Vite + TypeScript (`/webpage/`)
- **Backend**: Rust with Rocket framework (`/server/`)
- **Database**: PostgreSQL with SQLx
- **Auth**: Argon2 password hashing

## Development vs Production

- **Development**: Frontend (Vite dev server on localhost:5173) proxies `/api` requests to backend (localhost:8000)
- **Production**: Backend serves built React app from `../webpage/dist/` as static files, deployed in Docker on remote VM

## Development Commands

### Frontend (from `/webpage/` directory)
```bash
npm run dev         # Start Vite dev server on localhost:5173
npm run build       # Build for production (outputs to dist/)
npm run lint        # Run ESLint
npm run preview     # Preview production build
```

### Backend (from `/server/` directory)
```bash
cargo run           # Start Rocket server on localhost:8000
cargo check         # Type check without building
cargo build         # Build the project
cargo test          # Run tests
```

### Environment Setup
- Backend requires `DATABASE_URL` environment variable in `.env` file
- Frontend development server proxies `/api` requests to backend at localhost:8000

## Key Application Flow

1. **Static Files**: Rust backend serves built React app from `../webpage/dist/`
2. **API Routes**:
   - `/users` - Get all users
   - `/login` - User authentication
   - `/signup` - User registration
   - `/db` - Fetch zoning data
3. **CORS**: Custom CORS fairing allows localhost origins
4. **Authentication**: Session cookies set on successful login, users redirected to `/home`
5. **Password Security**: Argon2 hashing with salts via `HashedPassword` wrapper type

## Database Schema

Users table with `username` (email) and hashed `password` fields. The app includes a `zoning` table for housing data.

## Code Structure

### Backend (`server/src/`)
- `main.rs` - Rocket app setup, CORS, static file serving
- `routers/users.rs` - Authentication endpoints with email validation
- `database/users.rs` - User model with password verification

### Frontend (`webpage/src/`)
- `App.tsx` - Main app with React Router setup and LoginForm component
- `Home.tsx` - Protected home page shown after successful login
- Uses React hooks for state management and React Router for navigation
- API calls use `/api` prefix which proxies to backend during development

## Development Notes

- Frontend and backend run independently during development but communicate via `/api` proxy
- Both implement email validation (client + server side)
- Error handling includes specific HTTP status codes for different failure cases
- Session management via HTTP-only cookies with UUID session IDs
- Successful login redirects users to `/home` route
- Designed for Docker deployment on remote VM

---
> Source: [willocho/housing-webpage-login](https://github.com/willocho/housing-webpage-login) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-19 -->
