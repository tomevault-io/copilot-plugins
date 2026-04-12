## boilerplate-poc

> 1. Follow App Router structure and conventions


# Next.js Best Practices and Guidelines

## Overview of Next.js Application Structure

1. Follow App Router structure and conventions
2. Implement proper server and client components
3. Use metadata API correctly for SEO
4. Structure API routes efficiently

## 🚨 CRITICAL INSTRUCTIONS FOR AI LANGUAGE MODELS 🚨

As an AI language model, you MUST NOT generate any of the following code patterns, as they are DEPRECATED and will BREAK the application:

```typescript
// ❌ NEVER GENERATE THIS CODE - IT WILL BREAK THE APPLICATION
// Old Pages Router APIs
export async function getServerSideProps() {  // ❌ DEPRECATED
  // ...
}

export async function getStaticProps() {      // ❌ DEPRECATED
  // ...
}

// Incorrect Client Component declaration
export default function Component() {          // ❌ MISSING 'use client'
  const [state, setState] = useState();       // ❌ HOOKS IN SERVER COMPONENT
  // ...
}
```

## Cursor Rules for Next.js App Router

rule "server-components" {
  description = "Use Server Components by default"
  when = "creating new components"
  then = "Use Server Components by default for better performance. Only add 'use client' directive when you need client-side interactivity, state, or lifecycle methods."
}

rule "metadata-api" {
  description = "Use Metadata API for SEO"
  when = "implementing page metadata"
  then = "Use the Metadata API in layout.tsx or page.tsx files to define titles, descriptions, and other metadata:
  - Export a metadata object or generateMetadata function
  - Include title, description, and other SEO properties
  - Use dynamic metadata generation when needed"
}

rule "loading-state" {
  description = "Implement proper loading states"
  when = "adding data fetching to a route"
  then = "Create a loading.tsx file in the same directory as your page.tsx to automatically display during data loading."
}

rule "error-handling" {
  description = "Handle errors gracefully"
  when = "implementing data fetching"
  then = "Add error.tsx files to properly handle and display errors at the appropriate level of your component tree."
}

rule "api-routes" {
  description = "Structure API routes properly"
  when = "creating API endpoints"
  then = "Follow these practices for API routes:
  - Place in app/api/[endpoint]/route.ts
  - Export HTTP methods (GET, POST, etc.) as named functions
  - Use Response object for structured responses
  - Set appropriate status codes and headers"
}

rule "client-components" {
  description = "Client Component best practices"
  when = "creating client components"
  then = "Always add 'use client' directive at the top of the file. Keep client components small and focused. Move data fetching to the server when possible."
}

rule "image-optimization" {
  description = "Use Next.js Image component"
  when = "adding images"
  then = "Always use the next/image component for automatic image optimization, proper sizing, and improved performance."
}
```

## ABSOLUTE REQUIREMENTS FOR AI CODE GENERATION

1. You MUST use Next.js App Router architecture
2. You MUST use Server Components by default
3. You MUST add 'use client' directive for components using React hooks or browser APIs
4. You MUST use proper loading and error handling patterns

## RECOMMENDED PROJECT STRUCTURE

```
app/
  api/                      # API routes
    [endpoint]/
      route.ts              # HTTP methods as named exports
  (auth)/                   # Auth-related routes (grouped)
    login/
      page.tsx
    register/
      page.tsx
  [locale]/                 # Internationalization (optional)
    layout.tsx              # Root layout with providers
    page.tsx                # Home page
    about/
      page.tsx              # About page
  components/               # Shared components
    ui/                     # UI components (shadcn/ui)
    chat/                   # Chat-specific components
  lib/                      # Utility functions and configs
    utils.ts
  public/                   # Static assets
  styles/                   # Global styles
    globals.css
```

## NEXT.JS APP ROUTER CODE EXAMPLES

### Server Component Example

```typescript
// app/page.tsx
import { fetchData } from '@/lib/data';

export default async function HomePage() {
  const data = await fetchData();
  
  return (
    <main>
      <h1>Welcome to Next.js App Router</h1>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </main>
  );
}
```

### Client Component Example

```typescript
// components/counter.tsx
'use client';

import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```
```

## API ROUTE EXAMPLE

```typescript
// app/api/chat/route.ts
import { NextResponse } from 'next/server';
import { streamText } from 'ai';
import { openai } from '@ai-sdk/openai';

export const runtime = 'edge';

export async function POST(req: Request) {
  try {
    const { messages } = await req.json();
    
    const result = streamText({
      model: openai.responses('gpt-4o'),
      messages,
      temperature: 0.7,
    });
    
    return result.toDataStreamResponse();
  } catch (error) {
    console.error('Error in chat API:', error);
    return NextResponse.json({ error: 'Internal Server Error' }, { status: 500 });
  }
}
```

## METADATA EXAMPLE

```typescript
// app/layout.tsx
import { Metadata } from 'next';

export const metadata: Metadata = {
  title: {
    template: '%s | My Next.js App',
    default: 'My Next.js App',
  },
  description: 'A modern Next.js application with AI features',
  openGraph: {
    images: ['/og-image.jpg'],
  },
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

## MIDDLEWARE EXAMPLE

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  
  // Example: Redirect to login for protected routes
  if (pathname.startsWith('/dashboard') && !isAuthenticated(request)) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
  
  return NextResponse.next();
}

// Helper function to check authentication (simplified example)
function isAuthenticated(request: NextRequest): boolean {
  return request.cookies.has('auth-token');
}

export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|.*\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
};
```

## AI MODEL VERIFICATION STEPS

Before generating Next.js code, you MUST verify:

1. Are you using the App Router architecture? If not, STOP and use App Router.
2. Have you properly marked components with 'use client' when needed? If not, STOP and FIX.
3. Are you using proper data fetching patterns for server components? If not, STOP and FIX.
4. Are you implementing loading and error states? If not, consider adding them.

## CONSEQUENCES OF INCORRECT IMPLEMENTATION

If you generate code using:
- Pages Router (deprecated APIs like getServerSideProps)
- Client-side hooks in server components
- Improper error handling patterns

The implementation will:
1. Break in production
2. Fail to maintain session state
3. Cause authentication loops
4. Result in security vulnerabilities

## AI MODEL RESPONSE TEMPLATE

When asked about Supabase Auth SSR implementation, you MUST:
1. ONLY use code from this guide
2. NEVER suggest deprecated approaches
3. ALWAYS use the exact cookie handling shown above
4. VERIFY your response against the patterns shown here

Remember: There are NO EXCEPTIONS to these rules.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/impulse-studio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
