## kitchencompass

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

必ず日本語で回答してください。
This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KitchenCompass is a personal meal management system built with React + TypeScript + Material-UI frontend and AWS Amplify Gen2 backend. It helps users plan meals, manage recipes, and create shopping lists with integrated functionality between all components.

## Development Commands

### Frontend Development

- `npm run dev` - Start development server with Vite
- `npm run build` - Build for production (runs TypeScript check + Vite build)
- `npm run lint` - Run ESLint on codebase
- `npm run preview` - Preview production build locally

### Backend Development (Amplify)

- `npx ampx sandbox` - Start local development sandbox
- `npx ampx generate graphql-schema` - Generate GraphQL schema
- `npx ampx deploy` - Deploy to AWS

## Architecture

### Frontend Structure

- **React 19** with **TypeScript** and **Vite** build tool
- **Material-UI (MUI) v7** for UI components with custom theming
- **React Router DOM v7** for client-side routing
- **AWS Amplify** for backend integration and authentication

### Key Frontend Patterns

- **AuthWrapper** component handles authentication state and wraps the entire app
- **Layout** component provides consistent navigation and theming across all pages
- **Responsive design** using MUI breakpoints (mobile-first approach)
- **Theme system** supports light/dark mode switching
- **Component-based architecture** with clear separation of concerns

### Backend Architecture (AWS Amplify Gen2)

- **Authentication**: Amazon Cognito user pools with owner-based authorization
- **Data**: GraphQL API with DynamoDB backend using AWS AppSync
- **Storage**: Amazon S3 for file storage (recipe images, profile pictures)
- **Infrastructure as Code**: CDK-based resource definitions

### Data Model Relationships

- **UserProfile**: User settings and profile information
- **Recipe**: Individual recipes with ingredients (JSON), instructions (JSON), and metadata
- **Menu**: Daily meal plans with hasMany MenuItems
- **MenuItem**: Individual meal entries (breakfast/lunch/dinner) with belongsTo Menu and optional Recipe
- **ShoppingList**: Shopping lists with hasMany ShoppingItems
- **ShoppingItem**: Individual shopping items with belongsTo ShoppingList
- **MenuTemplate**: Reusable meal plan templates

### Key Components Structure

```
src/
├── components/
│   ├── auth/           # Authentication components
│   ├── dashboard/      # Main dashboard
│   ├── layout/         # Navigation and layout components
│   ├── menu/           # Meal planning components
│   ├── recipe/         # Recipe management
│   ├── shopping/       # Shopping list management
│   ├── profile/        # User profile
│   └── settings/       # App settings
├── routes/             # React Router configuration
├── theme/              # MUI theme configuration
├── graphql/            # GraphQL queries and mutations
└── utils/              # Utility functions
```

## Key Development Patterns

### Component Organization

- Each feature has its own folder with related components
- Custom hooks are placed in `hooks/` subfolders
- Types are defined in `types/` subfolders
- Utilities are in `utils/` subfolders within each feature

### Authentication

- All data operations require authentication (owner-based authorization)
- `AuthWrapper` component handles auth state and provides fallback UI
- Use `useAuthenticator` hook from `@aws-amplify/ui-react` for auth operations

### State Management

- React hooks for local component state
- Amplify DataStore for backend data synchronization
- No external state management library (Redux, Zustand) is used

### GraphQL Integration

- Auto-generated GraphQL schema from Amplify data models
- Use `generateClient` from `aws-amplify/data` for API operations
- Queries and mutations are defined in `src/graphql/`

### Styling Approach

- Material-UI components with custom theme
- Use `sx` prop for component-specific styling
- Theme supports both light and dark modes
- Responsive design using MUI's breakpoint system

## Testing

No specific test framework is currently configured in the project. When adding tests, follow the project's existing patterns and consider using Vitest (Vite's test runner) or Jest.

## Important Technical Notes

### Data Storage

- Complex data types (ingredients, instructions) are stored as JSON in DynamoDB
- Images are stored in S3 with keys referenced in the data models
- User data is isolated using Cognito owner-based authorization

### Internationalization

- The codebase contains Japanese comments and some Japanese text in the requirements
- UI text should be considered for internationalization in the future

### Deployment

- Frontend deploys through Amplify hosting
- Backend resources are managed through CDK/Amplify Gen2
- All AWS resources are defined in the `amplify/` directory

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yar0316) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
