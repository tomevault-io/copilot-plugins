## klattertradetnyhem

> This document provides guidelines for AI agents working with this codebase. This is a React therapy/mental health application with community features, built with TypeScript, Vite, and Tailwind CSS.

# AGENTS.md - AI Assistant Guide

This document provides guidelines for AI agents working with this codebase. This is a React therapy/mental health application with community features, built with TypeScript, Vite, and Tailwind CSS.

## Project Overview

**Technology Stack:**
- React 19 with TypeScript in strict mode
- Vite as build tool (development server on port 3000)
- Tailwind CSS v4 with custom design system
- Shadcn/ui component library
- Radix UI primitives
- Supabase for backend/authentication
- Three.js/React Three Fiber for 3D graphics
- Google Gemini AI integration

**Application Structure:**
- Main site: Therapy service landing pages
- Gemenskap: Community platform with chat features
- PWA support with install functionality

## Development Commands

```bash
# Development
npm run dev          # Start Vite dev server on port 3000
npm run build        # Build for production
npm run preview      # Preview production build

# Linting & Type Checking
# Note: No dedicated lint/typecheck scripts found - add them if needed
npx eslint .         # Run ESLint manually
npx tsc --noEmit     # Type check without emitting files
```

**Testing:** No testing framework currently configured. Consider adding Vitest or Jest.

## Code Style Guidelines

### Import Organization
```typescript
// External libraries (React, third-party)
import React, { useState, useEffect } from 'react';
import { Button } from '@/components/ui/button';

// Internal modules (sorted alphabetically)
import Navigation from './components/Navigation';
import { Page } from './public/types';
```

**Key Import Patterns:**
- Use `@/` prefix for absolute imports (configured in vite.config.ts and tsconfig.json)
- Group imports: React/Radix → External → Internal → Types
- Avoid relative imports when possible, prefer absolute paths

### Component Architecture
```typescript
// Functional components with TypeScript interfaces
interface ComponentProps {
  title: string;
  onPageChange?: (page: Page) => void;
}

const Component: React.FC<ComponentProps> = ({ title, onPageChange }) => {
  // Hooks at top level
  const [state, setState] = useState<string>('');
  
  // Event handlers
  const handleClick = () => {
    // Implementation
  };
  
  // Render
  return (
    <div className="container mx-auto px-6">
      {/* Component JSX */}
    </div>
  );
};

export default Component;
```

### TypeScript Patterns
- Use strict TypeScript (already configured)
- Define interfaces for all props and data structures
- Use enums for fixed sets of values
- Prefer `const` assertions for objects: `const config = { ... } as const;`

### Styling Conventions
```typescript
// Use cn() utility for conditional classes
import { cn } from '@/lib/utils';

const Component = ({ variant, className }) => (
  <div className={cn(
    'base-classes',
    variant === 'primary' && 'primary-classes',
    className
  )}>
```

**Design System:**
- **Colors:** Orange/amber primary (`#b35c2a`), dark backgrounds
- **Typography:** Inter for body text, Outfit for headers/display. Strictly avoid mono/serif fonts for UI elements.
- **Theme:** Dark/light mode support with CSS custom properties
- **Animations:** Custom keyframes in index.css (slideInUp, fade-in, bg-shift)

### File Naming & Structure
```
src/
├── components/          # Reusable UI components
│   ├── ui/             # Shadcn/ui components
│   └── gl/             # Three.js/3D components
├── pages/              # Page-level components
├── gemenskap/          # Community platform
│   ├── components/
│   ├── services/
│   └── types.ts
├── lib/                # Utilities (cn function, etc.)
└── types.ts            # Global type definitions
```

**Naming Conventions:**
- Components: PascalCase (e.g., `Navigation.tsx`)
- Files: camelCase for utilities, PascalCase for components
- Functions/variables: camelCase
- Constants: UPPER_SNAKE_CASE
- Interfaces/Types: PascalCase with descriptive names

### State Management
- Use React hooks for local state
- Custom hooks for complex logic (`usePWAInstall`, etc.)
- No global state management library detected (consider Context API)
- Props drilling is acceptable for this scale

### Error Handling
```typescript
// Async operations with proper error handling
const fetchData = async () => {
  try {
    const result = await someApiCall();
    setData(result);
  } catch (error) {
    console.error('Failed to fetch data:', error);
    // Show user-friendly error message
  }
};

// Type guards for runtime validation
const isValidProfile = (data: unknown): data is Profile => {
  return typeof data === 'object' && data !== null && 'id' in data;
};
```

### API Integration
- Supabase for database/auth
- Google Gemini AI for chat features
- Environment variables via Vite's `import.meta.env`
- Service files in `services/` directories

### Component Patterns
**Shadcn/ui Integration:**
```typescript
import * as React from "react";
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const buttonVariants = cva(/* ... */);
export interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement>, VariantProps<typeof buttonVariants> {
  // Props definition
}
```

**Form Handling:**
- React Hook Form with Zod validation
- Consistent form patterns across the application

## Development Guidelines

### Before Making Changes
1. Run `npm run dev` to ensure development server works
2. Check existing components for similar patterns
3. Follow established design system (colors, spacing, typography)

### Code Quality
- TypeScript strict mode enforced
- ESLint configured with Next.js rules (even though using Vite)
- No current testing setup - add tests for critical functionality

### Performance Considerations
- Use React.memo for expensive components
- Lazy load heavy components (3D graphics)
- Optimize bundle size with dynamic imports

### Security
- Never commit secrets or API keys
- Use environment variables for sensitive data
- Validate user inputs (Zod schemas available)

## Common Issues & Solutions

**Vite + Next.js Config Conflict:**
The project has both Vite and Next.js configurations. Use Vite for development/build, ignore Next.js configs.

**Path Aliases:**
- `@/` maps to project root
- Use absolute imports consistently

**Component Variants:**
Follow the CVA (class-variance-authority) pattern for component variants seen in Shadcn/ui components.

## Future Improvements

1. Add testing framework (Vitest recommended)
2. Add dedicated lint/typecheck npm scripts
3. Consider state management for larger features
4. Standardize error boundaries
5. Add storybook for component documentation

## Contact/Context

This is a mental health/therapy platform with community features. Maintain professional, accessible, and calming design patterns. The user experience should prioritize clarity and ease of use for potentially vulnerable users.

---
> Source: [KlattertradetAB/klattertradetnyhem](https://github.com/KlattertradetAB/klattertradetnyhem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
