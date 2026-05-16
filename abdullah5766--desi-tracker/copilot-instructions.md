## desi-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Desi-Tracker is a full-stack nutrition tracking application focused on South Asian/Desi foods. It consists of:
- **Backend**: Express.js REST API with Prisma ORM and PostgreSQL
- **Frontend**: React (Vite) with Tailwind CSS and Zustand for state management
- **Database**: PostgreSQL with Prisma schema defining Users, Foods, and FoodEntries

## Development Commands

### Backend (Root directory)
```bash
npm run dev          # Start development server with nodemon
npm start           # Start production server
npm run db:generate # Generate Prisma client
npm run db:push     # Push schema changes to database
npm run db:migrate  # Run database migrations
npm run db:studio   # Open Prisma Studio
npm run db:seed     # Seed database with initial data
```

### Frontend (frontend/ directory)
```bash
npm run dev     # Start Vite development server (port 5173)
npm run build   # Build for production
npm run lint    # Run ESLint
npm run preview # Preview production build
```

## Architecture

### Backend Structure
- **src/server.js**: Main Express server with middleware, CORS, rate limiting, and error handling
- **src/routes/**: API route handlers (auth, user, food)
- **src/middleware/auth.js**: JWT authentication middleware
- **prisma/schema.prisma**: Database schema with User, Food, FoodEntry models

### Frontend Structure
- **src/App.jsx**: Main routing logic with protected routes
- **src/stores/**: Zustand stores for auth, calories, and food state management
- **src/components/**: Reusable UI components including CaloriesCalculator, FoodTracker, NutritionOverview
- **src/pages/**: Dashboard and LandingPage components
- **src/services/api.js**: Axios API client with authentication headers

### Key Features
- JWT-based authentication with token persistence
- Food database with support for Desi/South Asian foods
- Calorie and macro tracking by meal type (BREAKFAST, LUNCH, DINNER, SNACK)
- User profile management with fitness goals and activity levels
- Rate limiting and security middleware (helmet, CORS)

### State Management
- **authStore.js**: Handles login, registration, token management, and user profile
- **calorieStore.js**: Manages daily calorie and nutrition tracking
- **foodStore.js**: Handles food search and management

### Database Schema
- **User**: Authentication, profile data, nutrition goals, activity level
- **Food**: Nutrition information, serving sizes, verification status, Desi food classification
- **FoodEntry**: Daily food logs linked to users and foods with meal types

## Environment Setup

Copy `.env.example` to `.env` and configure:
- PostgreSQL DATABASE_URL
- JWT_SECRET and JWT_EXPIRES_IN
- PORT (default 5000 for backend)
- FRONTEND_URL (default http://localhost:5173)
- RATE_LIMIT_MAX for API rate limiting

## Security Features

- JWT token authentication
- Password hashing with bcryptjs
- Helmet middleware for security headers
- CORS configuration
- Express rate limiting
- Input validation with express-validator
- Prisma for SQL injection protection

---
> Source: [Abdullah5766/Desi-Tracker](https://github.com/Abdullah5766/Desi-Tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
