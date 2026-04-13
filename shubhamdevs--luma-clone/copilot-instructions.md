## luma-clone

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Luma event management platform clone built with React and plain CSS. The application allows users to create and manage events with various features including public/private events, ticketing, and event themes.

## Architecture

The project follows a React-based architecture with:

- **Tech Stack**: React with plain CSS (no framework-specific styling)
- **Database**: Supabase (connection details in project_instructions.md)
- **Authentication**: Clerk authentication system
- **Folder Structure**: Component-based organization with separate folders for each component containing:
  - `<component>.jsx` - Main component file
  - `<component>.module.css` - Component-specific styles
  - `app.jsx` - Application entry point (where applicable)

## Key Features

### Event Management
- Event listing page showing all available events
- Create Event functionality with comprehensive options:
  - Public/Private event visibility
  - Event name and descriptions
  - Start and end date selection with calendar integration
  - Location support (offline locations and virtual links)
  - Event options including:
    - Ticketing (free or paid)
    - Approval requirements
    - Capacity management (numbered or unlimited)
  - Event themes (2-3 theme options available)

## Development Commands

- `npm install` - Install project dependencies
- `npm run dev` - Start development server (Vite) on port 3000
- `npm run build` - Build production version
- `npm run preview` - Preview production build
- `npm run lint` - Run ESLint for code quality
- `npm test` - Run Jest tests

## Environment Setup

### Required Environment Variables
Create a `.env` file based on `.env.example`:

```env
# Supabase Configuration
VITE_SUPABASE_URL=https://pdtsssifvmxunygrztgb.supabase.co
VITE_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InBkdHNzc2lmdm14dW55Z3J6dGdiIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTIwMjc0MDEsImV4cCI6MjA2NzYwMzQwMX0.9Vot-XRPm1UD5hQAdCZqJ0NT0SDf_A8tGIKPFqgcX1A

# Database
SUPABASE_DB_PASSWORD=iYKINbkT3e9K1BSx
SUPABASE_DB_URL=postgresql://postgres:iYKINbkT3e9K1BSx@db.pdtsssifvmxunygrztgb.supabase.co:5432/postgres
```

### Supabase Setup
- **Database**: PostgreSQL hosted on Supabase
- **Authentication**: Supabase Auth (no Clerk needed)
- **Project URL**: https://pdtsssifvmxunygrztgb.supabase.co
- **Database Password**: iYKINbkT3e9K1BSx
- **Client Library**: @supabase/supabase-js for all database operations and authentication

### Authentication
- **Supabase Auth**: Primary authentication method
- **User Management**: Built-in user registration, login, and session handling
- **Protected Routes**: Use Supabase auth state for route protection
- **Auth Flow**: Email/password authentication with Supabase built-in UI components

## Tech Stack Details

- **Build Tool**: Vite for fast development and optimized builds
- **Framework**: React 18 with functional components and hooks
- **Styling**: Plain CSS with CSS modules (no Tailwind/frameworks)
- **Database**: Supabase (PostgreSQL) with `@supabase/supabase-js`
- **Authentication**: Supabase Auth (built-in authentication system)
- **Routing**: React Router v6
- **Date Handling**: date-fns library for date manipulation
- **Testing**: Jest with React Testing Library

## Important Files

- `project_instructions.md` - Detailed feature requirements and UI specifications
- `chat.MD` - Step-by-step instruction log for development process
- `package.json` - Project dependencies and scripts
- `vite.config.js` - Vite configuration with React plugin
- `index.html` - Entry HTML file with proper meta tags
- `.env.example` - Environment variable template

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shubhamdevs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shubhamdevs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
