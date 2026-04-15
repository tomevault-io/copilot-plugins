## replicache-todo

> This is a Next.js 15 todo application built with Replicache for real-time synchronization and offline-first functionality.

# Replicache Todo App - Cursor Rules

## Project Context
This is a Next.js 15 todo application built with Replicache for real-time synchronization and offline-first functionality.

## Tech Stack
- Next.js 15 with App Router
- React 19
- TypeScript
- Replicache for real-time sync
- Tailwind CSS for styling
- shadcn/ui components
- Radix UI primitives
- Use pnpm for package management

## Architecture
- Replicache Provider wraps the entire app in `app/layout.tsx`
- Todo types are defined in `types/index.ts`
- Replicache configuration and hooks are in `lib/replicache-provider.tsx`
- All pages can access Replicache via `useReplicache()` and `useTodoMutations()` hooks

## Code Standards
- Use TypeScript with strict typing
- Prefer functional components with hooks
- Use "use client" directive for client components
- Import types with `import type` when possible
- Use async/await for Replicache mutations
- Do not generate any console.log statements unless it is explicitly requested
- prefer types over interfaces

## Replicache Patterns
- Access todos via `const { todos, loading } = useReplicache()`
- Use mutations via `const { createTodo, updateTodo, deleteTodo, toggleTodo } = useTodoMutations()`
- All mutations are optimistic and real-time
- Data structure: `Todo { id, text, completed, createdAt, updatedAt, deletedAt }`

## File Structure
- `/app` - Next.js app router pages
- `/lib` - Replicache provider and utilities
- `/types` - TypeScript type definitions
- `/components` - Reusable UI components (shadcn/ui)

## Environment Variables
- `NEXT_PUBLIC_REPLICACHE_LICENSE_KEY` - Required for Replicache

## Styling
- Uses Tailwind CSS with shadcn/ui components
- Dark theme support via CSS variables
- Responsive design patterns 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/konradmi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
