## k-k-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a private couple's web application built with Next.js 14+ (App Router) and Supabase. The app handles expense tracking, shared to-do lists, and calendar scheduling for exactly two authorized users.

## Technology Stack

- **Frontend**: Next.js 14+ (App Router), TypeScript, Tailwind CSS
- **UI Components**: shadcn/ui, Lucide React icons
- **Backend/Database**: Supabase (PostgreSQL + Auth + Storage)
- **Authentication**: Supabase Auth with Google OAuth
- **Date Handling**: date-fns
- **Calendar**: react-big-calendar or FullCalendar (planned)
- **PWA**: next-pwa (planned)
- **Development Tools**: Biome (linting/formatting)
- **Deployment**: Vercel

## Key Architecture Principles

### Authentication & Security
- Only two specific email addresses can register (configured via environment variables)
- Row-Level Security (RLS) policies restrict data access based on user relationships
- Expenses and schedules are shared between both users
- Individual schedule events can only be modified by their creator
- Shared schedule events can be modified by either user

### Database Schema
The application uses 5 main tables:
1. `profiles` - User profile data linked to Supabase auth.users
2. `expenses` - Financial transactions in Japanese Yen
3. `todo_categories` - Shared task categories
4. `todos` - Tasks assignable to self/partner/both
5. `schedules` - Calendar events (shared/individual types)

### Mobile-First Design
- Progressive Web App (PWA) optimized for smartphones
- Bottom tab navigation for main features
- Responsive design with mobile-first approach

## Core Features

### Expense Management
- Record expenses in Japanese Yen with date, description, amount, store, and payer
- Monthly expense views sorted by date (newest first)
- Settlement calculations showing who owes whom

### To-Do Lists
- Shared categories between users
- Tasks assignable to self, partner, or both
- Due date sorting (earliest first, no due date last)
- Any user can complete/edit/delete any task

### Schedule Sharing
- Events in JST timezone with start/end times
- Visual distinction between shared and individual events
- Location and description fields (optional)
- Intuitive calendar interface for adding events

## Development Notes

### Current State
This project is in the planning/documentation phase. The actual Next.js application has not been created yet. The current files are specification documents only.

### Next Steps (Phase 0)
1. Create Next.js project with TypeScript and Tailwind CSS
2. Set up Biome for linting/formatting
3. Configure Supabase integration
4. Implement authentication with restricted registration
5. Set up deployment pipeline with Vercel

### Data Access Patterns
- All database operations should use Supabase Server Actions
- Implement proper RLS policies for multi-user data sharing
- Currency amounts stored as integers (Japanese Yen)
- All timestamps should consider JST timezone for schedules

## 🔨 Most Important Rule - Process for Adding New Rules
When receiving instructions from users that seem to require ongoing application (not just one-time):

1. Ask "Should I make this a standard rule?"
2. If YES is received, add it as an additional rule to CLAUDE.md
3. Apply it as a standard rule going forward

This process allows for continuous improvement of project rules.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yken2257) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
