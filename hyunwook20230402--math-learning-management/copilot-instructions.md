## architecture

> Architecture and project structure guidelines for Math Mistake Master


# 🏗️ Architecture & Project Structure

## Core Architecture Principles

### Component Organization
- **Page Components**: Located in `src/pages/` organized by user role (`teacher/`, `student/`, `cms/`)
- **Reusable Components**: Located in `src/components/` with subdirectories for different types
- **UI Components**: Use shadcn/ui components from `src/components/ui/`
- **Custom Hooks**: Place in `src/hooks/` for reusable logic

### File Structure Rules
```
src/
├── components/          # Reusable UI components
│   ├── auth/           # Authentication components
│   ├── layout/         # Layout components (Header, etc.)
│   └── ui/             # shadcn/ui component library
├── hooks/              # Custom React hooks
├── integrations/       # External service integrations
├── lib/                # Utility libraries and API services
├── pages/              # Page components by user role
└── types/              # TypeScript type definitions
```

### Import Organization
Always organize imports in this order:
1. React and external libraries
2. Internal components (using `@/` alias)
3. API services and utilities
4. Type definitions

```typescript
// External libraries
import React from 'react';
import { useNavigate } from 'react-router-dom';

// Internal components
import { Button } from '@/components/ui/button';
import { useAuth } from '@/hooks/useAuth';

// API services
import { problemApi } from '@/lib/api';
```

## Database Architecture

### ID Usage Rules
- **Supabase Auth user.id**: Only for authentication, never as foreign key
- **profiles.id**: Use for all foreign key references
- **profiles.user_id**: Links to Supabase Auth user.id

### API Layer Structure
- All database operations go through `src/lib/api.ts`
- Organize APIs by domain: `profileApi`, `problemApi`, `problemSetApi`, etc.
- Always include proper error handling and logging

## State Management

### Global State
- Use `AuthContext` for authentication state
- Use React Query for server state management
- Keep component state local when possible

### Data Flow
```
User Action → API Call → Supabase → React Query Cache → Component Update
```

## Security Architecture

### Authentication
- Always check authentication state before rendering protected routes
- Use role-based access control (teacher/student)
- Implement proper route guards

### Data Security
- Leverage Supabase Row Level Security (RLS)
- Validate all inputs with Zod schemas
- Never expose sensitive data in client-side code

---
> Source: [hyunwook20230402/math-learning-management](https://github.com/hyunwook20230402/math-learning-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
