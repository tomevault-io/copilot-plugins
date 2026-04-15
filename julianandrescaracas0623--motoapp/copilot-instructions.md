## motoapp

> Reglas específicas para Next.js App Router.


# Next.js Rules

Reglas específicas para Next.js App Router.

## App Router

Este proyecto usa Next.js 16+ con App Router.

### File Structure

```
src/app/
├── layout.tsx          # Root layout
├── page.tsx            # Home page
├── globals.css         # Global styles
├── (auth)/
│   ├── login/
│   │   └── page.tsx
│   └── register/
│       └── page.tsx
├── api/
│   └── users/
│       └── route.ts
└── dashboard/
    ├── layout.tsx
    └── page.tsx
```

## Route Handlers (API)

```typescript
// src/app/api/users/route.ts
import { NextResponse } from "next/server";

export async function GET() {
  const users = await getUsers();
  return NextResponse.json({ users });
}

export async function POST(request: Request) {
  const body = await request.json();
  // ... validate and create
  return NextResponse.json({ user }, { status: 201 });
}
```

## Server Components por Defecto

Todos los archivos en `app/` son Server Components por defecto.

```tsx
// src/app/page.tsx - Server Component
async function getData() {
  // Fetch en el servidor
  return await db.query.users.findMany();
}

export default async function HomePage() {
  const users = await getData();
  return <UsersList users={users} />;
}
```

## Metadata

```tsx
import { Metadata } from "next";

export const metadata: Metadata = {
  title: "My App",
  description: "Description here",
};
```

## Dynamic Routes

```tsx
// src/app/users/[id]/page.tsx
interface PageProps {
  params: Promise<{ id: string }>;
}

export default async function UserPage({ params }: PageProps) {
  const { id } = await params;
  const user = await getUser(id);
  // ...
}
```

## Layouts Anidados

```tsx
// src/app/dashboard/layout.tsx
export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}
```

## Loading States

```tsx
// src/app/dashboard/loading.tsx
export default function Loading() {
  return <DashboardSkeleton />;
}
```

## Error Handling

```tsx
// src/app/dashboard/error.tsx
"use client";

export default function Error({
  error,
  reset,
}: {
  error: Error;
  reset: () => void;
}) {
  return (
    <div>
      <h2>Something went wrong</h2>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

## Runtime Selection

### Node.js (default)

```typescript
export const runtime = "nodejs"; // o omitir
```

### Edge Runtime

```typescript
export const runtime = "edge";
```

Usar Edge solo cuando:
- Latencia crítica
- No se necesitan APIs de Node.js
- Geolocation necesaria

## next.config.ts

Configuraciones clave:

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  // ... config
};

export default nextConfig;
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julianandrescaracas0623) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
