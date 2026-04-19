## mathquest

> **Live Application**: https://igor-kan.github.io/mathquest

# MathQuest - Interactive Mathematics Learning Platform

## Project Overview
**Live Application**: https://igor-kan.github.io/mathquest

Interactive mathematics learning platform featuring quest-based challenges, advanced progress tracking, and comprehensive gamification systems. Built with Next.js and deployed to GitHub Pages.

## Technology Stack
- **Framework**: Next.js with TypeScript
- **UI/UX**: Tailwind CSS, Radix UI, Lucide React icons
- **Data Visualization**: Recharts for progress analytics
- **Special Libraries**: 
  - `cmdk` - Command palette for quick navigation
  - `vaul` - Mobile-optimized modal components
- **Deployment**: GitHub Pages with static export

## Core Features

### 📊 Advanced Progress Tracking System
- **Real-time XP System**: Achievement unlocks and streak tracking
- **Performance Analytics**: Personalized learning metrics and accuracy calculations
- **Centralized Management**: Progress Tracker Singleton pattern for state management
- **Data Persistence**: Local storage with export functionality

### 🎯 Interactive Analytics Dashboard
- **Multi-tabbed Interface**: Progress, achievements, analytics, and leaderboard views
- **Achievement System**: 10+ categories with visual badges and completion tracking
- **Data Export**: Progress sharing and backup functionality
- **Visual Charts**: Multiple chart types using Recharts library

### 🧠 Enhanced Study Session Management
- **Real-time Tracking**: Session timer with exercise-by-exercise monitoring
- **Interactive Features**: Hint system and skip functionality with accuracy tracking
- **XP Calculation**: Bonus multipliers for streaks and accuracy performance
- **Session Analytics**: Time-per-exercise and performance metrics

### 📱 Modern Dashboard Interface
- **Quick Actions**: Daily/weekly goal cards with integrated study tips
- **Progress Visualization**: Comprehensive charts and performance indicators
- **Responsive Design**: Mobile-first approach optimized for all device sizes
- **Modern UI**: Clean, intuitive interface with smooth animations

## Key Components

### Core Libraries
- `lib/progress-tracker.ts` - Singleton pattern for centralized user progress management
- `components/analytics-dashboard.tsx` - Comprehensive analytics and visualization dashboard
- `components/study-session.tsx` - Interactive study session management with real-time tracking
- `app/dashboard/page.tsx` - Modern dashboard interface with quick actions and navigation

### Features Implemented (January 2025)
- ✅ Real-time XP tracking with level progression system
- ✅ Comprehensive achievement system with 10+ unlockable badges
- ✅ Global leaderboard with user ranking and competitive elements
- ✅ Session analytics with performance metrics and time tracking
- ✅ Data export functionality for progress sharing and backup
- ✅ Responsive design optimized for all device sizes

## Development Commands
```bash
npm run dev          # Start development server
npm run build        # Build for production
npm run export       # Generate static files for GitHub Pages
npm run lint         # Run ESLint
npm run type-check   # TypeScript type checking
```

## Deployment
- **Platform**: GitHub Pages
- **Build Process**: Static export with `npm run export`
- **URL**: https://igor-kan.github.io/mathquest
- **Auto-deployment**: GitHub Actions workflow

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/igor-kan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
