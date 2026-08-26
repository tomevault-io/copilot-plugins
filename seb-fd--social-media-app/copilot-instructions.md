## social-media-app

> npm run dev          # Start development server (localhost:3000)

# AGENTS.md - Social Media App

## Build & Development Commands

```bash
# Development
npm run dev          # Start development server (localhost:3000)
npm run build        # Build for production
npm run start        # Start production server

# Code Quality
npm run lint         # Run ESLint

# Database
npx prisma generate  # Generate Prisma client
npx prisma db push   # Push schema to database
npx prisma studio    # Open Prisma Studio

# Environment
cp .env.example .env # Copy environment template (if exists)
```

## Project Overview

This is a Next.js 14 social media application with:
- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript
- **Database**: PostgreSQL (Neon) with Prisma ORM
- **Auth**: Clerk
- **File Upload**: UploadThing
- **UI**: Tailwind CSS + shadcn/ui (Radix UI)

## Architecture

```
src/
├── app/                    # Next.js App Router (pages, layouts, API routes)
├── actions/                # Server Actions ("use server")
├── components/             # React components
│   ├── ui/                # shadcn/ui base components
│   └── *.tsx              # Feature components
├── lib/                    # Utilities and helpers
│   ├── prisma.ts          # Prisma singleton
│   ├── utils.ts           # cn() utility
│   ├── sanitize.ts        # XSS prevention
│   └── mentions.ts        # @mention extraction
├── middleware.ts           # Clerk middleware
└── types/                 # Shared TypeScript types
```

## Code Style Guidelines

### TypeScript

- Use explicit types for function parameters and return types
- Prefer `type` over `interface` for object shapes (see `src/types/index.ts`)
- Use `null` explicitly (not `undefined`) for optional fields
- Use optional chaining (`?.`) and nullish coalescing (`??`) appropriately

### React Components

**Server Components (default)**:
- No `"use client"` directive
- Fetch data directly using Server Actions or Prisma
- Props are serializable

**Client Components**:
- Add `"use client"` at the top of the file
- Use when: hooks, browser APIs, event handlers, interactive UI
- Keep client/server boundary clean - minimize client components

**Component Patterns**:
```tsx
// Client component with props interface
"use client";

import { cn } from "@/lib/utils";

type Props = {
  postId: string;
  hasLiked: boolean;
  canDelete?: boolean; // optional props last
};

export default function LikeButton({ postId, hasLiked, canDelete }: Props) {
  // component code
}
```

### Imports

**Order**:
1. React/core imports (`"use client"`, `React`)
2. Next.js imports (`next/navigation`, `next/image`)
3. Third-party libraries (`lucide-react`, `date-fns`)
4. Project imports (`@/components`, `@/actions`, `@/lib`)
5. Relative imports (`./`, `../`)

**Path aliases**: Use `@/` for src root (configured in tsconfig.json)

```tsx
import { useState } from "react";
import Image from "next/image";
import { HeartIcon } from "lucide-react";
import { cn } from "@/lib/utils";
import { likePost } from "@/actions/like.action";
import { Button } from "@/components/ui/button";
import "./styles.css";
```

### Server Actions

**File naming**: `*.action.ts`
**Directive**: `"use server"` at the top

```typescript
"use server";

import { prisma } from "@/lib/prisma";
import { getDbUserId } from "./user.action";
import { revalidatePath } from "next/cache";

export async function createPost(content: string, image: string) {
  try {
    const userId = await getDbUserId();
    if (!userId) return;

    const post = await prisma.post.create({
      data: { content, image, authorId: userId },
    });

    revalidatePath("/");
    return { success: true, post };
  } catch (error) {
    console.error("Failed to create post:", error);
    return { success: false, error: "Failed to create post" };
  }
}
```

**Error handling pattern**:
- Wrap in try/catch
- Log errors with `console.error`
- Return `{ success: false, error: "message" }` for user-facing errors
- Throw for critical errors that should halt execution

### Database (Prisma)

**Queries**: Use `select` to fetch only needed fields (avoid `include` when possible)

```typescript
const posts = await prisma.post.findMany({
  select: {
    id: true,
    content: true,
    author: {
      select: { id: true, username: true },
    },
  },
});
```

**Transactions**: Use `prisma.$transaction()` for atomic operations

```typescript
await prisma.$transaction([
  prisma.like.create({ data: { userId, postId } }),
  prisma.notification.create({ data: { ... } }),
]);
```

**Cascade deletes**: Use `onDelete: Cascade` in schema for related data cleanup

### UI Components (shadcn/ui)

**Button variants**:
```tsx
import { Button } from "@/components/ui/button";

// Variants: default, destructive, outline, secondary, ghost, link
<Button variant="destructive">Delete</Button>

// Sizes: default, sm, lg, icon
<Button size="sm">Small</Button>
```

**Utility function**: Use `cn()` from `@/lib/utils` for class merging

```tsx
import { cn } from "@/lib/utils";

<div className={cn("base-class", isActive && "active-class", className)} />
```

### Styling (Tailwind CSS)

- Use design system colors from `globals.css` (primary, secondary, destructive, etc.)
- Use `focus-visible` for keyboard accessibility
- Support dark mode using `dark:` prefix
- Keep classes sorted logically (positioning, then sizing, then styling)

### Security

**Input sanitization**: Use `sanitizeInput()` from `@/lib/sanitize`
```typescript
import { sanitizeInput, isValidUrl } from "@/lib/sanitize";

const sanitized = sanitizeInput(userInput);
```

**URL validation**: Always validate URLs before use
```typescript
if (!isValidUrl(website)) throw new Error("Invalid URL");
```

**Authorization**: Verify ownership before mutations
```typescript
if (post.authorId !== userId) throw new Error("Unauthorized");
```

### Naming Conventions

- **Files**: kebab-case (`like-button.tsx`, `user.action.ts`)
- **Components**: PascalCase (`LikeButton`, `CreatePost`)
- **Server Actions**: camelCase with descriptive verbs (`createPost`, `toggleLike`)
- **Types/Interfaces**: PascalCase (`type Post = {...}`)
- **Constants**: SCREAMING_SNAKE_CASE for config values

### State Management

- Use React hooks (`useState`, `useEffect`) for local state
- Use `useTransition` for async state updates
- Use `router.refresh()` from `next/navigation` for server state sync
- Avoid over-fetching; use Server Components for initial data

### Toast Notifications

Use `react-hot-toast` for user feedback:
```tsx
import toast from "react-hot-toast";
import { Loader2 } from "lucide-react";

// Loading state
const promise = toast.promise(apiCall(), {
  loading: "Saving...",
  success: "Saved!",
  error: "Failed to save",
});
```

### Date Formatting

Use `date-fns` with `formatDistanceToNow` for relative times:
```tsx
import { formatDistanceToNow } from "date-fns";
import { es } from "date-fns/locale";

formatDistanceToNow(new Date(createdAt), { addSuffix: true, locale: es })
```

### Icons

Use `lucide-react` for icons:
```tsx
import { HeartIcon, MessageCircleIcon, TrashIcon } from "lucide-react";
```

## Testing

The project does not currently have a test suite configured. If adding tests:
- Use **Vitest** for unit/integration tests
- Use **Playwright** for E2E tests
- Place tests alongside source files: `Component.tsx` → `Component.test.tsx`

## Git Workflow

```bash
# Create feature branch
git checkout -b feature/your-feature-name

# Commit (conventional commits not enforced but recommended)
git commit -m "feat: add new feature"

# Push
git push origin feature/your-feature-name
```

## Environment Variables

Required in `.env`:
```
DATABASE_URL=postgresql://...
CLERK_SECRET_KEY=sk_...
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
UPLOADTHING_SECRET=sk_...
UPLOADTHING_APP_ID=...
```

## Performance Notes

- Use `take + 1` pattern for cursor-based pagination
- Use `select` instead of `include` to reduce data transfer
- Optimize images with `next/image`
- Use skeleton loaders for loading states

## Common Tasks

**Add new Server Action**:
1. Create/edit `src/actions/*.action.ts`
2. Add `"use server"` at top
3. Export async function with error handling
4. Call `revalidatePath()` when modifying data

**Add new UI component**:
1. Use shadcn/ui: `npx shadcn@latest add component-name`
2. Or create in `src/components/ui/` following existing patterns

**Add database model**:
1. Update `prisma/schema.prisma`
2. Run `npx prisma db push`
3. Run `npx prisma generate`

**Add new route**:
1. Create folder in `src/app/`
2. Add `page.tsx` for content
3. Add `loading.tsx` for loading state
4. Add `error.tsx` for error handling

---
> Source: [Seb-fd/social-media-app](https://github.com/Seb-fd/social-media-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
