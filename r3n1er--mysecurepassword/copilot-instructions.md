## next-js-supabase-auth-ssr-guidelines

> description: Guidelines for developing Next.js apps with Supabase Auth SSR



---
description: Guidelines for developing Next.js apps with Supabase Auth SSR
globs: ["app/**", "lib/**", "middleware.ts"]
alwaysApply: true
---

# Next.js with Supabase Auth SSR Guidelines

## Overview
These rules enforce best practices for implementing Supabase Auth SSR in a Next.js 15 project using the App Router, ensuring secure and reliable authentication.

## Critical Instructions
- **MUST** use `@supabase/ssr` for all authentication-related code.
- **MUST** use `getAll` and `setAll` for cookie handling.
- **NEVER** use `get`, `set`, or `remove` cookie methods or import from `@supabase/auth-helpers-nextjs` (these are deprecated and will break the application).
- **MUST** verify all generated code against these rules before completion.

## Correct Browser Client Implementation
```typescript
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

## Correct Server Client Implementation
```typescript
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export async function createClient() {
  const cookieStore = await cookies()
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch {
            // Ignore errors in Server Components if middleware refreshes sessions
          }
        }
      }
    }
  )
}
```

## Correct Middleware Implementation
- Always include `supabase.auth.getUser()` to check user sessions.
- Protect routes by redirecting unauthenticated users to `/login`.
- Return `supabaseResponse` to maintain session integrity.
```typescript
import { createServerClient } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  let supabaseResponse = NextResponse.next({ request })
  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() { return request.cookies.getAll() },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value, options }) => request.cookies.set(name, value))
          supabaseResponse = NextResponse.next({ request })
          cookiesToSet.forEach(({ name, value, options }) =>
            supabaseResponse.cookies.set(name, value, options)
          )
        }
      }
    }
  )
  const { data: { user } } = await supabase.auth.getUser()
  if (!user && !request.nextUrl.pathname.startsWith('/login') && !request.nextUrl.pathname.startsWith('/auth')) {
    const url = request.nextUrl.clone()
    url.pathname = '/login'
    return NextResponse.redirect(url)
  }
  return supabaseResponse
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)']
}
```

## Verification Steps
Before generating code:
1. Ensure **ONLY** `getAll` and `setAll` are used for cookies.
2. Confirm imports are from `@supabase/ssr`.
3. Check for absence of `get`, `set`, or `remove` cookie methods.
4. Verify no imports from `@supabase/auth-helpers-nextjs`.

## Consequences of Incorrect Implementation
Using deprecated patterns will:
- Break session management in production.
- Cause authentication loops.
- Introduce security vulnerabilities.

---
> Source: [R3n1er/mysecurepassword](https://github.com/R3n1er/mysecurepassword) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
