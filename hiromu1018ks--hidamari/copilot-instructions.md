## hidamari

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Basic Policy

- Agents must think in English but always communicate with users in Japanese.

## Project Overview

Hidamari is a positive SNS application that uses Google's Gemini AI to analyze post content for positivity. This is a **greenfield project** - implementation follows the detailed plan in `docs/tasks.md`.

## Technology Stack

- **Frontend**: React 19 + Vite + TypeScript + Tailwind CSS + shadcn/ui
- **Backend**: Express.js + TypeScript + Prisma ORM
- **Database**: PostgreSQL (Docker)
- **AI**: Google Gemini API for positivity analysis
- **Authentication**: Auth.js with Google & GitHub OAuth

## Common Development Commands

Since this is a greenfield project, these commands will be available once implementation begins:

```bash
# Project setup (when implemented)
npm install                    # Install dependencies
docker-compose up -d          # Start PostgreSQL database
npx prisma migrate dev        # Run database migrations
npx prisma generate          # Generate Prisma client

# Development
npm run dev                   # Start development server
npm run build                # Build for production
npm run start                # Start production server

# Testing (when implemented)
npm test                     # Run tests
npm run test:watch          # Run tests in watch mode
npm run test:coverage       # Run tests with coverage

# Code quality (when implemented)
npm run lint                 # Run ESLint
npm run type-check          # Run TypeScript checks
```

## Architecture

This project uses a **separate frontend/backend architecture**:

```
Frontend (React + Vite) :5173
    ↓ HTTP API
Backend (Express.js) :3001
    ↓ API Calls
Gemini API (Google AI)
    ↓ Database
PostgreSQL (Docker) :5432
```

## Key Implementation Areas

### API Endpoints (Backend - Express.js)
- `POST /api/posts/analyze` - Gemini positivity analysis
- `POST /api/posts` - Create approved posts
- `GET /api/posts` - Paginated post feed
- `/api/auth/*` - Auth.js OAuth endpoints (signin, signout, callback)
- `GET /api/auth/session` - Get current session

### Core Components (Frontend - React + Vite)
- **PostForm** - Input with real-time analysis (shadcn/ui Textarea + Button)
- **PostFeed** - Infinite scroll post list (shadcn/ui Card)
- **PositivityIndicator** - Score display (shadcn/ui Progress + Badge)
- **AuthProvider** - Auth.js session management

### Services (Backend)
- **GeminiService** - AI analysis and suggestions
- **AuthService** - Auth.js session validation
- **PostService** - Database operations

## Data Models

```typescript
// Core interfaces
interface AnalysisResult {
  isPositive: boolean;
  score: number; // 0-100
  suggestion?: string;
  reason: string;
}

interface Post {
  id: string;
  content: string;
  positivityScore: number;
  author: User;
  createdAt: Date;
}
```

## Environment Setup

```bash
# Backend (.env)
DATABASE_URL="postgresql://postgres:password@localhost:5432/positive_sns"
AUTH_SECRET="your-auth-secret"
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"
GITHUB_CLIENT_ID="your-github-client-id"
GITHUB_CLIENT_SECRET="your-github-client-secret"
GEMINI_API_KEY="your-gemini-key"

# Frontend (.env)
VITE_API_URL="http://localhost:3001"
```

## Project Structure

This project follows a **separate frontend/backend monorepo structure**:

```
Hidamari/
├── docs/                    # Project documentation
├── frontend/               # React + Vite frontend
├── backend/                # Express.js backend
├── shared/                 # Common types and utilities
└── scripts/                # Development scripts
```

### Key Directories (when implemented)
- **frontend/src/components/feature/**: auth/, posts/ (feature-based components)
- **backend/src/services/**: GeminiService, AuthService, PostService
- **backend/src/middleware/**: auth, validation, errorHandler, rateLimit
- **shared/types/**: Common TypeScript interfaces

## Development Workflow

### Initial Setup (when implemented)
```bash
# 1. Start PostgreSQL
docker-compose up -d

# 2. Backend setup
cd backend
npm install
npx prisma migrate dev
npm run dev

# 3. Frontend setup (separate terminal)
cd frontend
npm install
npm run dev
```

### Testing Commands (when implemented)
```bash
# Frontend tests
cd frontend && npm test              # Vitest + React Testing Library
cd frontend && npm run test:watch   # Watch mode

# Backend tests
cd backend && npm test              # Jest + Supertest
cd backend && npm run test:unit     # Unit tests only
cd backend && npm run test:integration # Integration tests only
```

## Key Constraints

- **Character Limit**: 1000 characters per post
- **API Timeout**: 10 seconds for Gemini API calls
- **Debouncing**: 500ms for real-time analysis
- **Security**: OAuth authentication, input validation with Zod
- **Testing Target**: 80%+ coverage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hiromu1018ks)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hiromu1018ks)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
