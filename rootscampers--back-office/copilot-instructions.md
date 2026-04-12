## back-office

> Next.js 15 App Router patterns for Rootscampers


# Next.js 15 App Router

## Summary

Implements Next.js 15 App Router with locale-based structure (`app/[lng]/`). Defines conventions for Server/Client Components, layouts, metadata, and API routes. Reference [data-validation](mdc:.cursor/rules/data-validation.mdc) for input validation in APIs, [security](mdc:.cursor/rules/security.mdc) for headers and security, and [styling](mdc:.cursor/rules/styling.mdc) for component styling.

## Route structure

All pages are under `app/[lng]/` to support i18n:

```
app/[lng]/
├── page.tsx              # Home
├── layout.tsx            # Layout with providers
├── camper-hire/[id]/     # Camper details
├── my-trips/             # My trips
└── auth/                 # Authentication
```

## Server vs Client Components

```tsx
// ✅ Server Component (default) - For data fetching
// app/[lng]/camper-hire/[id]/page.tsx
export default async function CamperPage({ params }: Props) {
  const camper = await getCamper(params.id);
  return <CamperDetails camper={camper} />;
}

// ✅ Client Component - Only when you need interactivity
// components/camper-hire/CamperCalendar.tsx
("use client");
import { useState } from "react";
export function CamperCalendar() {
  const [dates, setDates] = useState(null);
  // ...
}
```

## File conventions

| File            | Usage                  |
| --------------- | ---------------------- |
| `page.tsx`      | Public route           |
| `layout.tsx`    | Shared layout          |
| `loading.tsx`   | Skeleton while loading |
| `error.tsx`     | Error boundary         |
| `not-found.tsx` | Custom 404             |

## Middleware

The middleware in `middleware.ts` handles:

- Language detection (cookie `i18next` → Accept-Language → fallback `es`)
- Redirect to routes with locale

```ts
// ❌ DON'T: Hardcode language
redirect("/es/camper-hire");

// ✅ DO: Use locale from context
redirect(`/${lng}/camper-hire`);
```

## Metadata and SEO

```tsx
// app/[lng]/camper-hire/[id]/page.tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const camper = await getCamper(params.id);
  return {
    title: camper.name,
    description: `Rent ${camper.name}...`,
  };
}
```

## API routes

API routes are in `app/api/` (without locale):

- `app/api/contact/` - Contact form
- `app/api/sitemap/` - Sitemap generation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RootsCampers)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/RootsCampers)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
