## bppay

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start Vite development server with hot reload
- `npm run build` - Build production bundle
- `npm run lint` - Run ESLint on all JavaScript/JSX files
- `npm run preview` - Preview production build locally

## Project Architecture

This is a React prototyping application built with Vite, designed for creating and managing product prototypes with collaborative features.

### Tech Stack
- **Frontend**: React 19 with JSX, React Router DOM for navigation
- **Styling**: Tailwind CSS with custom configuration
- **Backend**: Supabase for database and authentication
- **Build Tool**: Vite with React plugin
- **Icons**: Lucide React

### Directory Structure
- `src/components/prototypes/` - Prototype-specific components including menu, layout, and individual prototypes
- `src/components/shared/` - Shared components like CommentSystem, UserProfile, BackButton
- `src/hooks/` - Custom React hooks, primarily for Supabase integration
- `src/lib/` - External service clients (Supabase configuration)
- `src/data/` - Static data definitions for prototypes

### Application Flow
The app follows a two-level routing structure:
1. **Prototype Menu** (`/` or `/prototypes`) - Main dashboard showing all available prototypes with search, filtering, and grid/list views
2. **Individual Prototypes** (`/prototypes/{id}`) - Specific prototype views wrapped in `PrototypeLayout` component

### Key Components
- `PrototypeMenu` - Main dashboard with prototype cards, search functionality, status filtering
- `PrototypeLayout` - Wrapper layout for individual prototypes with navigation and context
- `CommentSystem` - Collaborative commenting system integrated with Supabase
- `useSupabaseComments` hook - Manages all comment CRUD operations

### Data Management
- Prototype metadata stored in `src/data/prototypes.js` as static configuration
- Comments and user-generated content stored in Supabase with real-time capabilities
- Environment variables required: `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY`

### Supabase Integration
- Comments table with fields: prototype_id, text, author, tab, created_at
- Real-time comment updates and collaborative features
- Error handling for missing environment variables

## Code Conventions

- Uses ES modules and modern React patterns (functional components, hooks)
- Tailwind CSS for styling with utility-first approach
- Lucide React for consistent iconography
- ESLint configuration with React hooks and refresh plugins
- Custom ESLint rule: unused variables starting with uppercase or underscore are ignored

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevnat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
