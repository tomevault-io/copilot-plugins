## file-uploader

> This is a PERN-TS (PostgreSQL, Express, React, Node.js, TypeScript) monorepo for file uploading, organizing, and sharing.


# File Uploader Project Structure

This is a PERN-TS (PostgreSQL, Express, React, Node.js, TypeScript) monorepo for file uploading, organizing, and sharing.

## Architecture Overview

- **Backend**: Express.js API with Prisma ORM, JWT auth, Stripe billing
- **Frontend**: React 19 + Vite + Tailwind CSS + Supabase storage
- **Database**: PostgreSQL with Prisma migrations
- **File Storage**: Supabase for file uploads/downloads
- **Authentication**: JWT with refresh tokens

## Directory Structure

- `backend/` - Express API server
  - `src/` - TypeScript source code
  - `prisma/` - Database schema and migrations
  - `dist/` - Compiled JavaScript output
- `frontend/` - React application
  - `src/` - React components, hooks, pages, utils
  - `dist/` - Built static assets
- Root level contains shared configuration

## Key Entry Points

- Backend: [backend/src/index.ts](mdc:backend/src/index.ts)
- Frontend: [frontend/src/main.tsx](mdc:frontend/src/main.tsx)
- Database Schema: [backend/prisma/schema.prisma](mdc:backend/prisma/schema.prisma)

## Development Ports

- Backend API: http://localhost:4000
- Frontend: http://localhost:5173

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rubengpr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
