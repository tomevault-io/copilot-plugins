## navi-ai

> 🚀 This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

🚀 This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Next.js 15 application** implementing **Feature Sliced Design (FSD)** architecture with **Supabase authentication**. It's designed as a scalable AI tools platform demonstrating modern React patterns with comprehensive authentication flows.

**Key Technologies:**
- Next.js 15 (App Router)
- React 19
- TypeScript 5.3.3
- Supabase (Authentication & Database)
- Tailwind CSS
- shadcn/ui components

**Package Manager:** pnpm (always use pnpm for all package operations)

## 💬 Communication Preferences

- 📌 **Summarize** – Briefly summarize each completed task
- 📏 **Change Scale** – Classify changes as `Small`, `Medium`, or `Large`
- ❓ **Clarify First** – Ask questions if any instruction is unclear
- 🧠 **Emoji Check** – Start each response with a random emoji to confirm context
- ⚠️ **Respect Emotion** – If urgency is signaled (e.g., "This is critical"), increase caution and precision

### Advanced Communication
- 📘 For `Large` changes, always plan → explain → wait for approval → execute
- 🧾 Always state what is complete vs. what is pending

## 🧑‍🏭 Coding Preferences

### Code Quality Principles
- **Simplicity**: Prioritize the simplest solution over complex ones
- **Avoid Redundancy**: Prevent code duplication and reuse existing functionality (**DRY principle**)
- **Guardrails**: Do not use mock data in development or production environments, except in tests
- **Efficiency**: Optimize output to minimize token usage while maintaining clarity

## 🔧 Technical Stack

**Required Stack:**
- **Package Manager**: pnpm (always use pnpm for installing libraries)
- **Frontend**: Next.js (App Router only)
- **Styling**: Tailwind CSS (no CSS-in-JS libraries like styled-components)
- **State Management**: React Hooks, fetch for server state
- **Database & Auth**: Supabase only
- **Date Utils**: date-fns
- **UI Components**: shadcn/ui

**Forbidden Technologies:**
- **No** additional backend frameworks (e.g., Express.js, Firebase)
- **No** CSS-in-JS (e.g., styled-components, emotion)
- **No** external state management libraries (e.g., Redux, Zustand)

## Development Commands

```bash
# Start development server
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Run linting
pnpm lint

# Add new shadcn/ui component
pnpm add-component [component-name]
```

## Core Architecture: Feature Sliced Design (FSD)

The project follows FSD methodology with these layers (from lowest to highest):

### **shared/** - Reusable Infrastructure
- UI components (following shadcn/ui patterns)
- Supabase client configurations
- Utility functions and constants
- TypeScript path: `@/shared/*`

### **entities/** - Business Entities
- Core business models (user, tool, category, review, subscription)
- Read-only domain logic
- UI components that represent entities
- TypeScript path: `@/entities/*`

### **features/** - Product Features
- Business functionality implementation (auth, login, sign-up, profile, review, compare, search, onboarding, subscription)
- CRUD operations and server actions
- State management logic
- TypeScript path: `@/features/*`

### **widgets/** - UI Composition Blocks
- Complex UI blocks composed from features, entities, and shared components
- Layout and presentation logic
- TypeScript path: `@/widgets/*`

### **app/** - Application Layer
- Next.js App Router pages and routing
- Global layouts and middleware configuration
- Combines FSD traditional "app" layer with "pages" layer
- TypeScript path: `@/app/*`

## Key Dependency Rules

1. **Unidirectional imports:** Higher layers can import from lower layers only
   - app → widgets → features → entities → shared
2. **Layer isolation:** Same-layer slices cannot depend on each other
3. **Public API:** Each slice exposes functionality through `index.ts`

## File Organization Patterns

Each FSD slice follows consistent structure:
```
slice-name/
├── ui/           # React components
├── model/        # Business logic, hooks, types
├── api/          # External API calls
├── action/       # Server actions (alternative to api/)
└── index.ts      # Public exports
```

### Naming Conventions
- **Components**: PascalCase
- **Hooks**: lowerCamelCase starting with 'use'
- **Types/Interfaces**: PascalCase with `.types.ts` suffix
- **Utilities**: kebab-case

## Authentication Architecture

### Supabase Integration
- **Client-side:** `@/shared/utils/supabase/client.ts`
- **Server-side:** `@/shared/utils/supabase/server.ts`
- **Middleware:** `@/shared/utils/supabase/middleware.ts`

### Auth Flow Patterns
- Server Actions are used for auth operations (look for `"use server"` directive)
- Session management through Next.js middleware
- Route protection via Supabase middleware integration

### Key Auth Components
- `@/features/auth/ui/AuthButton.tsx` - Authentication state display
- `@/features/login/ui/LoginForm.tsx` - Login form implementation
- `@/features/sign-up/ui/SignUpForm.tsx` - Registration form
- `@/features/auth/ui/ResetPasswordButton.tsx` - Password reset

### Authentication State Checking

**Server Components:**
```tsx
import { getIsAuthenticated } from "@packages/auth/src/features";

export default async function ProtectedPage() {
  const isAuthenticated = await getIsAuthenticated();

  if (!isAuthenticated) {
    redirect(`${getOrigin()}${AUTH_PATHNAME}${SIGN_IN_PATHNAME}`);
  }

  return <div>Protected Content</div>;
}
```

**Client Components:**
```tsx
"use client";
import { useAuth } from "@packages/auth/src/hooks";

export default function AuthButton() {
  const { signOut } = useAuth();
  return <button onClick={signOut}>Sign Out</button>;
}
```

## Component Development Patterns

### Adding UI Components
1. **shadcn/ui components:** Use `pnpm add-component [name]` - automatically configured to use `@/shared/ui`
2. **Custom components:** Place in appropriate FSD layer (`shared/ui`, `entities/*/ui`, `features/*/ui`, `widgets/*/ui`)

### Styling Conventions
- **Primary:** Tailwind CSS utility classes only
- **Configuration:** Uses "new-york" shadcn/ui style with zinc base color
- **CSS Variables:** Enabled for theming
- **Global styles:** Located in `src/app/globals.css`

### TypeScript Patterns
- **Strict mode** enabled
- **Path mapping** configured for all FSD layers
- **No generic for useState of primary type**

## State Management Patterns

### Client State
- Use React hooks for local state
- Shared state through context when needed
- **No external state management libraries**

### Server State
- Use server actions for mutations
- Fetch data in server components when possible
- Cache appropriately using Next.js patterns

## Server Actions & Data Fetching

### Server Actions Pattern
Server actions are located in `action/` folders within appropriate FSD slices:
- Authentication: `@/features/login/action/login.ts`, `@/features/sign-up/action/create-user.ts`
- Profile management: `@/features/profile/action/updateUser.ts`
- Reviews: `@/features/review/action/` (CRUD operations)

### Data Flow
- **Server Components:** Fetch data directly using Supabase server client
- **Client State:** React hooks for local state management
- **Server State:** Server actions for mutations, direct fetching in server components
- **No external state management:** Project avoids Redux, Zustand, etc.

## Environment Setup

Required environment variables:
```bash
NEXT_PUBLIC_SUPABASE_URL=your-supabase-url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-supabase-anon-key
```

## Route Structure

The project uses Next.js App Router with route groups:
- `(auth)/` - Authentication pages (login, sign-up, reset-password)
- `(main)/` - Main application pages with shared layout
- Parallel routes: `@review-drawer/` for modal overlays
- Dynamic routes: `[slug]/` for tool pages

## Middleware Configuration

Authentication middleware runs on all routes except:
- Static files (`_next/static`, `_next/image`)
- Image files (svg, png, jpg, jpeg, gif, webp)
- favicon.ico

This ensures proper session management across the application.

## Development Best Practices

### Performance
- Use Next.js Image component patterns
- Implement proper loading states
- Use date-fns for date utilities

### Code Organization
- Follow FSD dependency rules strictly
- Each layer should have clear public API through `index.ts`
- Avoid direct cross-layer dependencies

### Quality Assurance
- Use TypeScript strictly
- Follow Tailwind utility-first approach
- Implement proper error boundaries
- Test authentication flows thoroughly

## Task Master AI Instructions
**Import Task Master's development workflow commands and guidelines, treat as if import is in the main CLAUDE.md file.**
@./.taskmaster/CLAUDE.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pec-camp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
