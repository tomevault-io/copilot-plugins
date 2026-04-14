## code-a-thon

> Welcome to the **Academic Pairing Hub**! This document serves as the primary instructional context for AI agents and developers working on this project.

# 🎓 Academic Pairing Hub - Developer Guide (GEMINI.md)

Welcome to the **Academic Pairing Hub**! This document serves as the primary instructional context for AI agents and developers working on this project.

## 🏗️ Project Overview

The **Academic Pairing Hub** is a modern, real-time platform designed to intelligently pair academic mentors and mentees. It focuses on research interests, department alignment, and professional relationship management.

### Core Tech Stack
- **Frontend**: React 19 (Vite), TypeScript (Strict Mode)
- **Styling**: Tailwind CSS v4 (using `@tailwindcss/vite`), Lucide Icons
- **Animations**: Framer Motion (via `motion` package)
- **Authentication**: Clerk (Primary Auth Provider)
- **Backend & Database**: Supabase (PostgreSQL, Realtime, Strict RLS)
- **Routing**: React Router DOM (v7)

### Architecture
The application follows a clean separation of concerns:
- **UI Components**: Located in `src/components/`, focused on presentation.
- **Pages**: Located in `src/pages/`, defining the main application views.
- **Logic & Data**: Handled via custom hooks in `src/hooks/` (e.g., `useSupabase`, `useMessages`, `usePairings`).
- **Services**: Found in `src/services/`, handling specific business logic or external API integrations.
- **Database**: Managed via `supabase/migrations/` and `src/types/database.types.ts`.

## 🚀 Building and Running

### Prerequisites
- Node.js 18+
- Supabase CLI (optional, but recommended for local dev)
- Clerk account and project configuration

### Key Commands
- `npm install`: Install dependencies.
- `npm run dev`: Start the development server at `http://localhost:3000`.
- `npm run build`: Create a production build in the `dist/` directory.
- `npm run lint`: Run TypeScript type checking (`tsc --noEmit`).
- `npm run clean`: Remove the `dist/` build directory.
- `npm run preview`: Preview the production build locally.

### Environment Variables
Required in `.env`:
- `VITE_SUPABASE_URL`: Your Supabase Project URL.
- `VITE_SUPABASE_ANON_KEY`: Your Supabase Anonymous Key.
- `VITE_CLERK_PUBLISHABLE_KEY`: Your Clerk Publishable Key.

## 🛠️ Development Conventions

### Security & Authentication
- **Clerk-Supabase Integration**: We use a custom `useSupabase` hook that intercepts fetch requests and injects a Clerk JWT (`session.getToken({ template: 'supabase' })`) into the `Authorization` header.
- **Strict RLS**: All Supabase tables MUST have Row Level Security enabled. Policies should validate against `auth.jwt()->>'sub'` to ensure users only access their own data. See `supabase/migrations/002_strict_rls.sql`.

### Styling & UI
- **Tailwind CSS v4**: Use utility classes for all styling. Ensure consistent use of the design system colors (e.g., `primary`, `surface`).
- **Animations**: Use `motion` (Framer Motion) for page transitions and interactive elements. Prefer staggered animations for lists and cards.
- **Icons**: Use `lucide-react` for all iconography.

### State Management & Data Fetching
- **Custom Hooks**: Business logic and data fetching should live in hooks. Do not put complex data fetching logic directly inside components.
- **Realtime**: Use Supabase Realtime for messaging and live updates (see `useMessages.ts`).

### TypeScript
- **Strict Mode**: The project enforces strict TypeScript checking. Always provide proper types and avoid `any`.
- **Database Types**: Update `src/types/database.types.ts` whenever the Supabase schema changes.

## 📈 Future Implementation: Onboarding Flow
Refer to `implementation_plan.md` for details on the upcoming 3-step onboarding wizard (Role Selection -> Profile Quiz -> Match Display). This will involve creating:
- `user_profiles` table in Supabase.
- `OnboardingGuard` component to protect main app routes.
- `matchingService.ts` for client-side pairing scores.

---
*This file is maintained by Gemini CLI. Update it as the project architecture evolves.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AliAlfridawi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AliAlfridawi)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
