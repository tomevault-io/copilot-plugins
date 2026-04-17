## hackathon-ranking-dashboard

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
AIハッカソン運営支援アプリ - チーム管理、いいね投票、ランキング機能を提供するNext.js Webアプリケーション

## Build & Development Commands
```bash
# Install dependencies
npm install

# Run development server
npm run dev

# Production build
npm run build

# Start production server
npm start

# Run tests
npm test

# Lint code
npm run lint

# Format code
npm run format
```

## High-Level Architecture

### Technology Stack
- **Frontend/Backend**: Next.js (App Router) with TypeScript
- **Authentication**: MSAL.js (Microsoft/Azure AD single tenant)
- **Database**: Azure Cosmos DB for NoSQL
- **Hosting**: Azure App Service (Node 18 LTS)
- **Monitoring**: Azure App Insights

### Data Flow Architecture
```
Browser (MSAL.js) ── HTTPS ── Next.js API Routes ── Azure Cosmos DB
                       │                   ↑
                       └─ Auth(JWT verify) ─┘
```

### Core Design Principles
- **Authentication**: All pages require MSAL sign-in (ID token verification with `jose`)
- **Authorization**: Three roles - Admin (UPN allowlist), Presenter (team member), Participant
- **Like System**: Atomic increment of `likeCount` in Cosmos DB, only allowed during team presentation
- **Polling**: 1-second polling for current presentation, no polling for rankings (snapshot-based)
- **Rate Limiting**: 20 req/s per user for like API

### Database Schema (Cosmos DB)

**teams container** (Partition key: `/eventId`):
```json
{
  "id": "<uuid>",
  "type": "team",
  "eventId": "default",
  "name": "Team Name",
  "orderIndex": 1,
  "isActive": true,
  "likeCount": 0,
  "display": {
    "title": "発表タイトル",
    "description": "概要",
    "demoUrl": "https://...",
    "repoUrl": "https://...",
    "tags": ["AI","Web"]
  },
  "members": [
    { "name": "名前", "email": "email@example.com" }
  ]
}
```

**settings container** (Partition key: `/eventId`):
- `currentPresentation`: Currently presenting team ID
- `rankingSnapshot`: Published ranking snapshot

### API Endpoints Structure
- `POST /api/likes` - Increment like (only during presentation)
- `GET /api/ranking-public` - Get published ranking snapshot
- `GET/POST /api/presentation/current` - Current presentation management
- Team CRUD operations under `/api/teams/`
- `PATCH /api/teams/:id/display` - Presenter can update display info
- `POST /api/draw` - Lottery for presentation order (Fisher-Yates shuffle)

### User Role Capabilities
- **Admin**: Full team/member management, lottery, presentation control, ranking publish
- **Presenter**: Edit own team's display information only
- **Participant**: View teams, like during presentations, view published rankings

### Environment Variables Required
```
AZURE_AD_TENANT_ID
AZURE_AD_CLIENT_ID
AZURE_AD_ISSUER
COSMOS_ENDPOINT
COSMOS_KEY
COSMOS_DB
COSMOS_CONTAINER_TEAMS
COSMOS_CONTAINER_SETTINGS
ADMIN_UPNS (comma-separated admin UPNs)
```

## Code Organization Guidelines
- Follow existing patterns in AGENTS.md
- TypeScript strict mode, 2-space indent, single quotes, semicolons
- Components: PascalCase, hooks: useXxx prefix
- API routes: kebab-case under app/api/
- Keep modules small and single-purpose
- Use dependency injection for lib code

## Implementation Status
- Design documents completed (docs/01-05_*.md)
- Implementation not yet started (apps/ directory is empty)
- Next steps: Initialize Next.js project and implement based on program design (docs/04_program_design.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sat0tabe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
