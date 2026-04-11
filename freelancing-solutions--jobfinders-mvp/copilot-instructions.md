## jobfinders-mvp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Server & Development
- `npm run dev` - Start development server with nodemon (runs on port 3010)
- `npm run build` - Build for production (includes Prisma generation)
- `npm start` - Start production server (uses environment PORT)
- `npm run lint` - Run ESLint

### Database Operations
- `npm run db:push` - Push schema changes to database
- `npm run db:generate` - Generate Prisma client
- `npm run db:migrate` - Run database migrations
- `npm run db:reset` - Reset database to initial state
- `npm run db:seed` - Seed database with initial data

### Testing
- `npm test` - Run Jest tests
- `npm run test:watch` - Run tests in watch mode
- `npm run test:coverage` - Run tests with coverage report

## Architecture Overview

This is a modern AI-powered job board platform built with Next.js 15, TypeScript, and Prisma.

### Core Technologies
- **Next.js 15** with App Router
- **TypeScript** with strict mode enabled
- **Prisma ORM** with PostgreSQL
- **NextAuth.js** for authentication
- **Socket.IO** for real-time features
- **Tailwind CSS 4** + shadcn/ui for styling
- **Zustand** for state management
- **TanStack Query** for data fetching

### Custom Server Architecture
- Uses custom `server.ts` that integrates Next.js with Socket.IO
- Development runs on port 3010, production uses environment PORT
- Nodemon handles hot reloading in development

### AI & Services Integration
- OpenAI GPT-4 integration for AI features
- TensorFlow.js for client-side ML processing
- Redis for AI response caching
- Custom ML models for job matching and ATS features

### Authentication & Roles
- NextAuth.js configured with custom roles: `seeker`, `employer`, `admin`
- Role-based access control throughout the application
- Password reset functionality with token verification

### Database Schema
The Prisma schema includes comprehensive models for:
- User management with role-based profiles
- Job posting and application tracking
- Resume/CV management with related entities
- Company and billing management
- AI session tracking and analytics

### Project Structure
```
src/
├── app/              # Next.js App Router pages & API routes
├── components/       # Reusable React components
│   └── ui/          # shadcn/ui components
├── ai/              # AI service integrations
├── hooks/           # Custom React hooks
└── lib/             # Utilities and configurations
prisma/
└── schema.prisma    # Database schema
docs/               # Architecture documentation
```

### Development Notes
- Path aliases configured: `@/*` maps to `src/*`
- TypeScript ignores build errors (configured in next.config.ts)
- ESLint ignores build errors for faster development
- Custom socket.io integration at `/api/socketio`
- Static file serving for uploads via rewrites

### Testing Setup
- Jest with React Testing Library
- jsdom environment configured
- Coverage reporting available
- Watch mode for active development

### Build Configuration
- TypeScript build errors are ignored (configured for rapid development)
- React strict mode disabled for development
- Webpack hot module replacement disabled (handled by nodemon)
- ESLint errors ignored during builds

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/freelancing-solutions)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/freelancing-solutions)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
