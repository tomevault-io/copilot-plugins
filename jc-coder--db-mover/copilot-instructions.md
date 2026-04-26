## db-mover

> This document outlines the architecture, patterns, and conventions of the DB Mover project to assist AI agents in understanding and contributing to the codebase.

# Agent Mode - DB Mover

This document outlines the architecture, patterns, and conventions of the DB Mover project to assist AI agents in understanding and contributing to the codebase.

## Project Overview

DB Mover is a full-stack application designed to facilitate the relocation of enterprise datasets across different cloud architectures. It supports multiple database ecosystems including MongoDB, PostgreSQL, BigQuery, and Redis.

## Tech Stack

### Frontend (`/client`)

- **Framework**: React with Vite
- **Styling**: Tailwind CSS
- **Animations**: Framer Motion
- **UI Components**: Radix UI (base for shadcn/ui components)
- **Icons**: Lucide React
- **State Management**: React hooks
- **API Communication**: Axios

### Backend (`/server`)

- **Runtime**: Node.js
- **Framework**: Express (implied by `index.ts`)
- **Database**: MongoDB (for job management/metadata)
- **Language**: TypeScript

## Architecture & Patterns

### Frontend

- **Components**: Functional components using Tailwind for styling.
- **UI System**: Located in `client/src/components/ui`, following the shadcn/ui pattern.
- **Pages**: Main application logic starts in `App.tsx`, with `LandingPage.tsx` as the entry point.
- **Styling Conventions**: Uses CSS variables for theming (`index.css`) and custom utility classes.
- **Naming Convention**: Interfaces start with `I` (e.g., `ILandingPageProps`). _Note: Currently some interfaces might not follow this, but new ones MUST._

### Backend

- **Services**: Business logic is encapsulated in service files (e.g., `server/src/services/migration.ts`).
- **Lib**: Shared utilities and library configurations (e.g., `server/src/lib/mongo.ts`, `server/src/lib/jobManager.ts`).
- **Entry Point**: `server/src/index.ts`.

## Coding Standards

- **TypeScript**: No `any` type allowed.
- **Interfaces**: All interfaces must start with `I` (e.g., `IUser`, `IDataSource`).
- **DRY Principle**: Check for existing helpers/utilities before implementing new ones.
- **State Management**: Keep it simple with hooks unless complexity requires more.

## Background & Styling

- **Theme**: Dark mode by default.
- **Backgrounds**: Uses a combination of grid patterns (`bg-grid`) and radial glows (`hero-glow`) defined in `index.css`.
- **Transitions**: Smooth animations using Framer Motion.

## Recent Changes & Focus

- Enhancing the visual appeal of the landing page with better background elements.
- Maintaining structural integrity during database migrations.

---
> Source: [JC-Coder/db-mover](https://github.com/JC-Coder/db-mover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
