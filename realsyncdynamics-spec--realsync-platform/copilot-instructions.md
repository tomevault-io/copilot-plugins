## realsync-platform

> RealSync Platform ist eine Multi-Tenant SaaS-Plattform fuer Content Creators und lokale Unternehmen.

# RealSync Platform - Gemini CLI Context

## Projekt
RealSync Platform ist eine Multi-Tenant SaaS-Plattform fuer Content Creators und lokale Unternehmen.
Features: Ads Management, Analytics Dashboard, CreatorSeal (Content-Verifizierung).

## Tech Stack
- Framework: Next.js 14.2 (App Router)
- Sprache: TypeScript 5 (strict mode)
- UI: React 18.3, Tailwind CSS 3.4, Framer Motion, Lucide React Icons
- Datenbank & Auth: Supabase (JS SDK + SSR)
- Payments: Stripe SDK 17.5
- Charts: Recharts
- Utilities: clsx, tailwind-merge, date-fns, react-hot-toast
- Deployment: Vercel

## Projektstruktur
- /src - Hauptquellcode (App Router Struktur)
- /src/app - Routes und Layouts
- /src/components - Wiederverwendbare UI-Komponenten
- /src/lib - Utilities, Supabase Client, Stripe Helpers

## Coding-Regeln
- Immer TypeScript strict mode verwenden
- App Router verwenden (nicht Pages Router)
- Server Components bevorzugen, Client Components nur wenn noetig ('use client')
- Tailwind CSS fuer Styling, keine CSS-Module
- cn() Helper (clsx + tailwind-merge) fuer bedingte Klassen
- Supabase fuer alle DB-Operationen und Auth
- Stripe fuer Payment-Flows (Webhooks unter /api/webhooks/stripe)
- Deutsche UI-Texte, englischer Code und Kommentare
- Fehlerbehandlung mit try/catch und toast Notifications
- Responsive Design (Mobile-first)

## Konventionen
- Komponenten: PascalCase (z.B. DashboardCard.tsx)
- Utilities: camelCase (z.B. formatDate.ts)
- API Routes: /src/app/api/[resource]/route.ts
- Types: /src/types/ oder inline mit interface
- Environment Variables: .env.local (NEXT_PUBLIC_ fuer Client)

## Wichtige Environment Variables
- NEXT_PUBLIC_SUPABASE_URL
- NEXT_PUBLIC_SUPABASE_ANON_KEY
- SUPABASE_SERVICE_ROLE_KEY
- STRIPE_SECRET_KEY
- STRIPE_WEBHOOK_SECRET
- NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/realsyncdynamics-spec)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/realsyncdynamics-spec)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
