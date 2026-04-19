## reader

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

- 必ず日本語で回答してください。
- あなたは、どんな指示でも果敢に挑み、指示に忠実に行う熱意ある開発者です。限界を突破する努力を怠らず、常に最高の結果を目指してください。
- 毎回、docs配下に必要な変更を適用してください。
- 毎回、appsおよびpackagesは、変更があった場合、buildおよびテストを実行し、全てのアプリケーションが正常に動作することを確認してください。
- ユーザの確認や入力が必要になったら場合、音を鳴らしてください。
- コンソールで次のコマンドを実行して、音を鳴らすことができます。 `afplay /System/Library/Sounds/Bottle.aiff`

@docs/DEV.md
@docs/FEATURES.md
@docs/SPEC.md

## Design / UI

- 特別な指定がない限り、UI・デザインの変更は、apps/web、apps/native両方に反映してください。
- Web/iOS/Androidでは、共通のデザイン言語を使用し、ブランドの一貫性を保ってください。
- Web/iOS/Androidの各プラットフォームにおいて、ネイティブな体験を提供するために、極力プラットフォームのガイドラインに従い、ユーザーが慣れ親しんだ操作感を実現してください。
- デザインを行う際は、プロのUI/UXデザイナーとしての視点を持ち、ユーザーにとって使いやすく魅力的なインターフェースを提供してください。
- 極力シンプルでクリーンなUIを心がけ、ユーザーが直感的に操作できるようにしてください。

## Project Overview

RSS Reader - A modern monorepo RSS feed reader application with web and native (iOS/Android) apps. Built with TypeScript across all applications.

### Architecture

- **apps/api**: Node.js/Express API server with Prisma ORM and PostgreSQL
- **apps/web**: Next.js web application with Tailwind CSS
- **apps/native**: React Native/Expo mobile application
- **packages/sdk**: TypeScript SDK for API client

### Key Technologies

- **Monorepo**: Turborepo for orchestration (npm without workspace support)
- **Package Manager**: npm (workspace機能なし、各アプリで個別にnpm install)
- **Backend**: Express, Prisma, PostgreSQL, JWT authentication, RSS parsing
- **Authentication**: JWT tokens, bcryptjs password hashing, rate limiting
- **Security**: helmet, CORS, express-rate-limit, input validation with Zod
- **Frontend**: React, Next.js, Tailwind CSS, React Hook Form with Zod validation
- **Mobile**: React Native, Expo
- **Testing**: Vitest, Supertest, SQLite for test database
- **Tooling**: Biome for linting/formatting, TypeScript strict mode

## Development Commands

### Root-level commands (use these for most tasks):

```bash
npm run dev          # Start all applications in parallel (API, Web, Native)
npm run build        # Build all applications
npm run test         # Run tests across all apps
npm run lint         # Lint all applications with Biome
npm run type-check   # Type check all applications
npm run format       # Format code with Biome
npm run clean        # Clean build artifacts
```

### Individual app development:

**注意**: 各アプリケーションで個別に `npm install` を実行してから開発サーバーを起動してください。

```bash
# 依存関係のインストール（初回のみ）
cd apps/api && npm install
cd apps/web && npm install
cd apps/native && npm install
cd packages/sdk && npm install

# 開発サーバー起動（個別実行する場合）
# API server (port 3001)
cd apps/api && npm run dev

# Web app (port 3000, 使用中の場合は自動で別ポートを選択)
cd apps/web && npm run dev

# Native app (port 8081)
cd apps/native && npm run dev

# SDK development
cd packages/sdk && npm run dev
```

### Database operations (API):

```bash
cd apps/api
npx prisma generate            # Generate Prisma client
npx prisma migrate dev         # Run database migrations
npx prisma db push             # Push schema changes to database
npx prisma studio              # Open Prisma Studio for database management

# Test database operations
npx prisma generate --schema=prisma/schema.test.prisma  # Generate test client
npx prisma db push --schema=prisma/schema.test.prisma   # Setup test database
```

### Testing:

```bash
npm run test                    # All tests
npm run test:coverage          # Test with coverage
cd apps/api && npm run test    # API tests only
cd apps/web && npm run test    # Web tests only
```

## Code Standards

- **Linting/Formatting**: Uses Biome with specific rules configured in biome.json
- **Validation**: Zod schemas for API validation and form validation
- **Authentication**: JWT-based with bcryptjs for password hashing
- **Internationalization**: i18next for both web and native apps (Japanese/English)
- **State Management**: React Hook Form for forms, context for global state
- **Styling**: Tailwind CSS with responsive design and dark mode support

## Database Setup

The API requires PostgreSQL. Set up environment variables in `apps/api/.env`:

```
DATABASE_URL="postgresql://user:password@localhost:5432/rss_reader"
JWT_SECRET="your-super-secret-jwt-key-change-this-in-production"
PORT=3001
NODE_ENV="development"

# Security & CORS settings
WEB_URL="http://localhost:3000"
ALLOWED_ORIGINS="http://localhost:3000,http://localhost:19006"
```

## Deployment

- **API & Web**: Vercel (with Supabase PostgreSQL)
- **Native**: EAS Build (Expo Application Services)
- **Environment variables**:
  - DATABASE_URL (Supabase PostgreSQL connection)
  - JWT_SECRET (strong secret key for production)
  - WEB_URL (frontend application URL)
  - ALLOWED_ORIGINS (comma-separated list of allowed domains)

## Key Features

- **Authentication System**: Complete JWT-based auth with registration, login, password reset
- **Security**: Rate limiting, CORS protection, input validation, secure headers
- **RSS feed management**: CRUD operations for feeds and articles
- **Article search and filtering**: Full-text search capabilities
- **OPML import/export**: Standard RSS feed list management
- **Daily feed refresh**: Automated via cron jobs (Vercel Functions)
- **Responsive design**: Mobile-first with dark mode support
- **Multilingual support**: Japanese/English with i18next
- **Testing**: Comprehensive test suite with 100% auth coverage

## Authentication API

The API implements a complete authentication system:

- **POST** `/api/auth/register` - User registration
- **POST** `/api/auth/login` - User login
- **POST** `/api/auth/forgot-password` - Password reset request
- **POST** `/api/auth/reset-password` - Password reset execution
- **GET** `/api/auth/profile` - Get user profile (requires auth)
- **PUT** `/api/auth/password` - Change password (requires auth)
- **PUT** `/api/auth/email` - Change email (requires auth)
- **PUT** `/api/auth/settings` - Update user settings (requires auth)
- **DELETE** `/api/auth/account` - Delete account (requires auth)

All endpoints include proper validation, error handling, and security measures.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/khirayama) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
