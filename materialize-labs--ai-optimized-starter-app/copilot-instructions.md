## auth

> Follow these rules when working on auth.

# Authentication Architecture

This document outlines our authentication approach using Clerk. Following these guidelines ensures consistent, secure user authentication throughout the application.

## Core Principles

1. **Security First**: Implement proper authentication checks for all protected resources
2. **Seamless Experience**: Provide smooth authentication flows with minimal friction
3. **Consistent Patterns**: Use standardized approaches for auth across the application
4. **Explicit Permissions**: Make access controls clear and intentional

## Clerk Integration

Our application uses [Clerk](mdc:https:/clerk.dev) for authentication, which provides:

- User management and authentication
- Social login providers
- Session management
- User profile management
- Security features like MFA

## Server-Side Authentication

### Importing Auth Helpers

Always use the correct imports for server components and server actions:

```tsx
// In server components
import { auth } from "@clerk/nextjs/server"

// In middleware or other server contexts
import { getAuth } from "@clerk/nextjs/server"
```

### Usage in Server Components

When accessing authentication in server components:

```tsx
// app/dashboard/page.tsx
import { auth } from "@clerk/nextjs/server"
import { redirect } from "next/navigation"

export default async function DashboardPage() {
  const { userId } = auth()
  
  if (!userId) {
    return redirect("/sign-in")
  }
  
  // Continue with authenticated page rendering
  return (
    <div>
      <h1>Dashboard</h1>
      {/* Dashboard content */}
    </div>
  )
}
```

### Usage in Server Actions

When using authentication in server actions, always await the auth helper:

```tsx
// actions/notes.ts
"use server"

import { auth } from "@clerk/nextjs/server"
import { db } from "@/db/db"
import { notesTable } from "@/db/schema/notes"
import { ActionState } from "@/types/server-action"

export async function createNote(
  content: string
): Promise<ActionState<{ id: string }>> {
  try {
    // Get authenticated user
    const { userId } = await auth()
    
    if (!userId) {
      return {
        isSuccess: false,
        message: "Authentication required"
      }
    }
    
    // Create note for authenticated user
    const [note] = await db.insert(notesTable)
      .values({ content, userId })
      .returning({ id: notesTable.id })
    
    return {
      isSuccess: true,
      message: "Note created successfully",
      data: { id: note.id }
    }
  } catch (error) {
    console.error("Failed to create note:", error)

    return {
      isSuccess: false,
      message: "Failed to create note"
    }
  }
}
```

## Client-Side Authentication

### Current User in Client Components

To access the current user in client components:

```tsx
"use client"

import { useUser } from "@clerk/nextjs"

export function UserProfile() {
  const { user, isLoaded } = useUser()
  
  if (!isLoaded) {
    return <div>Loading...</div>
  }
  
  if (!user) {
    return <div>Not signed in</div>
  }
  
  return (
    <div>
      <h2>Profile</h2>
      <p>Hello, {user.firstName}!</p>
      <img 
        src={user.imageUrl} 
        alt="Profile" 
        className="w-16 h-16 rounded-full"
      />
    </div>
  )
}
```

## Protecting Routes

### Route Protection with Middleware

Configure `middleware.ts` at the root of your project:

```ts
// middleware.ts
import { authMiddleware } from "@clerk/nextjs"

export default authMiddleware({
  // Public routes that don't require authentication
  publicRoutes: [
    "/",
    "/sign-in",
    "/sign-up",
    "/pricing",
    "/api/webhook",
    "/blog(.*)", // Support for wildcards
  ],
})

export const config = {
  matcher: ["/((?!_next/static|_next/image|favicon.ico).*)"],
}
```

---
> Source: [materialize-labs/ai-optimized-starter-app](https://github.com/materialize-labs/ai-optimized-starter-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
