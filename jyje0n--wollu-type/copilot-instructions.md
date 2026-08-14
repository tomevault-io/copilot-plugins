## wollu-type

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Requirements

- **Node.js**: >=20.0.0 (required for cross-env@10.0.0 and other dependencies)
- **Yarn**: >=1.22.0

## Development Commands

- `yarn dev` - Start development server with Turbopack
- `yarn build` - Build the project for production with Turbopack
- `yarn start` - Start production server
- `yarn lint` - Run ESLint for code quality checks

## Database Commands

The project uses Prisma with PostgreSQL:

- `npx prisma generate` - Generate Prisma client after schema changes
- `npx prisma db push` - Push schema changes to database
- `npx prisma migrate dev` - Create and apply database migrations
- `npx prisma studio` - Open Prisma Studio for database management

## Architecture Overview

### Core Structure

This is a **Next.js 15 타이핑 연습 애플리케이션**으로 한국어와 영어 타이핑을 지원합니다.

**Key Technologies:**
- Next.js 15 with App Router and Turbopack
- TypeScript 
- Tailwind CSS v4
- Prisma ORM with PostgreSQL
- Zustand for state management
- Framer Motion for animations
- Material-UI components

### Application Structure

**Data Flow:**
- `src/stores/` - Zustand stores for global state (settings, user data)
- `src/components/core/TypingEngine.tsx` - Main typing logic and keyboard handling
- `src/components/core/TextRenderer.tsx` - Text display and visual feedback
- `src/data/` - Static text data (words/sentences for Korean/English)
- `src/utils/aiAdvice.ts` - AI-powered typing analysis and feedback

**Database Models (Prisma):**
- `User` - User profiles and authentication
- `UserPreferences` - Typing settings and theme preferences  
- `TypingRecord` - Individual typing session results
- `Sentence/Word` - Text content for practice

**Key Features:**
- Real-time typing accuracy calculation
- Multiple text types (words vs sentences)
- Configurable difficulty settings
- Ghost mode for reduced visual distractions
- AI-powered performance analysis
- Tier system for skill progression

### Important Patterns

**Settings Management:**
Settings are stored in Zustand with local storage persistence. All typing configuration happens through `useSettingsStore()`.

**Typing State:**
The core typing logic is in `TypingEngine.tsx` which handles:
- Keyboard event capture
- Real-time accuracy calculation  
- Timer and statistics tracking
- Text generation based on user settings

**Data Persistence:**
Typing results are automatically saved to the database via `/api/typing-records` endpoint after each session completion.

**Language Support:**
The app supports both Korean (CPM-based) and English (WPM-based) typing metrics with appropriate calculations for each language.

---
> Source: [JYJE0N/wollu-type](https://github.com/JYJE0N/wollu-type) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
