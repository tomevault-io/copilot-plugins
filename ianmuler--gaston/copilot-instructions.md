## gaston

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GastOn is a React Native expense tracking mobile application built with:
- **Frontend**: React Native + TypeScript with Expo Router (file-based routing)
- **State Management**: TanStack Query for server state management
- **Backend**: Express.js + PostgreSQL (planned)
- **Architecture**: Three main screens - Weekly View, Manage, and Balances

## Development Commands

### Mobile App (React Native with Expo)
```bash
cd mobile
npm start                    # Start Expo development server
npm run android             # Run on Android device/emulator
npm run ios                 # Run on iOS device/simulator  
npm run web                 # Run in web browser
npm test                    # Run Jest tests
npm run lint                # Run ESLint
npm run type-check          # Run TypeScript compiler check
```

## Project Architecture

### File Structure
```
GastOn/
├── mobile/                              # React Native App with Expo
│   ├── app/                            # Expo Router - file-based routing
│   │   ├── (tabs)/                     # Tab group layout
│   │   │   ├── _layout.tsx             # Tab navigation layout
│   │   │   ├── index.tsx               # Weekly View (main screen)
│   │   │   ├── manage.tsx              # Categories and expense names management
│   │   │   └── balances.tsx            # Balance dashboard and grouped view
│   │   ├── _layout.tsx                 # Root layout
│   │   └── +not-found.tsx              # 404 screen
│   ├── src/
│   │   ├── components/                 # Reusable React components
│   │   ├── hooks/                      # TanStack Query custom hooks
│   │   ├── services/                   # API client and service functions
│   │   ├── types/                      # TypeScript type definitions
│   │   ├── utils/                      # Utility functions and constants
│   │   └── styles/                     # Design system (colors, spacing, typography)
│   ├── package.json
│   └── tsconfig.json                   # TypeScript configuration with path mappings
├── base/                               # Project documentation
│   ├── Plan_1.md                       # Detailed implementation plan
│   └── Proyecto.md                     # Original project specifications
└── backend/                            # Express.js API (planned)
```

### TypeScript Path Mappings
The project uses TypeScript path mappings for cleaner imports:
- `@/*` → `src/*`
- `@/components/*` → `src/components/*`  
- `@/styles/*` → `src/styles/*`
- `@/types/*` → `src/types/*`
- `@/utils/*` → `src/utils/*`
- `@/services/*` → `src/services/*`

### Key Components and Architecture

#### Data Models
- **Expense**: Core entity with amount, date, category, and expense name
- **Category**: Expense categories with colors (Food, Transport, Entertainment, etc.)
- **ExpenseName**: Predefined expense names linked to suggested categories
- **Balance**: Calculated totals and statistics for dashboard views

#### Navigation Structure (Expo Router)
- Tab-based navigation with three main screens
- File-based routing system
- Layout components for consistent UI structure

#### State Management (TanStack Query)
- Server state caching and synchronization
- Custom hooks for expenses, categories, and names
- Automatic refetching and cache invalidation

#### Design System
- Color palette adapted from timeTracker project
- Consistent spacing and typography system
- Category-specific colors for visual organization

## Data Architecture

### Core Entities
```typescript
// Expense with full details
interface ExpenseWithDetails {
  id: number;
  monto: number;
  fecha: string; // YYYY-MM-DD
  categoria: Category;
  nombre_gasto: ExpenseName;
  descripcion?: string;
}

// Weekly organization
interface WeeklyData {
  weekStart: string;
  weekEnd: string;
  days: DayExpenses[];
  weekTotal: number;
}
```

### Database Schema (PostgreSQL)
- `categorias`: Expense categories with colors
- `nombres_gastos`: Predefined expense names with suggested categories  
- `gastos`: Individual expense records with foreign keys

## Screen Functionality

### Weekly View (index.tsx)
- Main screen showing expenses organized by days of the week
- Current week display with day-based tabs
- Tap to edit expenses inline
- Floating action button to add new expenses

### Manage Screen (manage.tsx)  
- Two tabs: Categories and Expense Names management
- CRUD operations for both categories and expense names
- Color picker for categories
- Cascading updates when modifying linked data

### Balances Screen (balances.tsx)
- Toggle between Dashboard and Grouped List views
- Dashboard: Summary cards with totals by category and name
- Grouped List: Collapsible sections by month and category
- Real-time calculations and updates

## Development Guidelines

### Adding New Features
1. Define TypeScript interfaces in `src/types/`
2. Create API service functions in `src/services/`
3. Build custom hooks with TanStack Query in `src/hooks/`
4. Implement UI components following existing design patterns
5. Use the established color system and spacing constants

### Code Organization
- Follow existing file structure and naming conventions
- Use TypeScript path mappings for imports
- Implement proper error handling in API calls
- Maintain consistent styling with the design system

### Testing and Quality
- Run `npm run type-check` before commits
- Use `npm run lint` to ensure code quality
- Test on both iOS and Android when possible
- Follow React Native and Expo best practices

## Reference Documents
- `base/Plan_1.md`: Comprehensive implementation plan with detailed specifications
- `base/Proyecto.md`: Original project requirements and user stories
- Design inspiration from adjacent timeTracker project

## Database Configuration
The app connects to a Neon PostgreSQL database. Environment configuration should be set up in `.env` file with proper connection parameters.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IanMuler) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
