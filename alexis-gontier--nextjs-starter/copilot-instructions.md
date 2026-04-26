## nextjs-starter

> Instructions pour Claude Code sur ce projet.

# CLAUDE.md

Instructions pour Claude Code sur ce projet.

## Stack

- Next.js 16 App Router + Turbopack, React 19, TypeScript 5
- Tailwind CSS v4, shadcn/ui (new-york), Radix UI, Lucide icons
- pnpm, ESLint 9 (flat config), Prettier, Husky + lint-staged + commitlint
- Alias : `@/*` → `./src/*`
- Langue HTML : `lang="fr"`

## Conventions de code

- Pas de point-virgule, guillemets doubles, indentation 2 espaces, trailing commas
- Les classes Tailwind sont triées automatiquement par prettier-plugin-tailwindcss
- Utiliser `cn()` depuis `@/lib/utils/cn` pour combiner des classes Tailwind
- Toujours importer `env` depuis `@/env` — jamais `process.env` directement
- Composants shadcn/ui installés dans `src/components/shadcn-ui/` via `pnpm dlx shadcn add <component>`

### Zod 4

Les formats de string sont des fonctions top-level en Zod 4 :

```ts
// ✅ Zod 4
z.email()
z.uuid()
z.url()

// ❌ Zod 3 (obsolète)
z.string().email()
z.string().uuid()
```

- Utiliser `.extend()` à la place de `.merge()` pour les ZodObject
- `z.record()` requiert deux arguments : `z.record(z.string(), z.number())`

### Next.js 16 — APIs async

`params`, `searchParams`, `cookies()`, `headers()`, `draftMode()` sont async :

```ts
// Page component
export default async function Page({
  params,
}: {
  params: Promise<{ id: string }>
}) {
  const { id } = await params
}

// Server Component
const cookieStore = await cookies()
const headersList = await headers()
```

### Next.js 16 — proxy.ts

`middleware.ts` est déprécié — utiliser `proxy.ts` à la place :

```ts
// proxy.ts (remplace middleware.ts)
export default function proxy(request: NextRequest) {
  return NextResponse.redirect(new URL("/home", request.url))
}
```

### Next.js 16 — Cache

`revalidateTag()` requiert un 2e argument (profil cacheLife) :

```ts
revalidateTag("tag", "max") // SWR — pour Server Components/Route Handlers
updateTag("tag") // read-your-writes — pour Server Actions uniquement
```

## Structure des fichiers

```
src/
  app/(app)/          — pages de l'app (route group)
  components/
    layout/           — composants de mise en page
    shadcn-ui/        — composants shadcn/ui
  lib/utils/cn.ts     — utilitaire cn()
  providers/          — providers React (theme, etc.)
  styles/             — globals.css + fonts.ts
  test/
    mocks/
      handlers.ts     — handlers MSW globaux
      server.ts       — serveur MSW Node.js
    utils/
      render.tsx      — render custom avec providers
  env.ts              — variables d'environnement typées
e2e/
  pages/              — Page Objects (un par page)
  *.spec.ts           — specs Playwright
```

## Testing — règles strictes

### Décision Vitest vs Playwright

```
Fonction / hook / composant Client synchrone  →  Vitest (.test.ts dans src/)
Async Server Component / navigation / flow    →  Playwright E2E (e2e/)
```

Les async RSC **ne peuvent pas** être testés avec Vitest — uniquement Playwright.

### Vitest

- Les `.test.ts(x)` sont **co-localisés** avec leur fichier source dans `src/`
- Toujours importer depuis `@/test/utils/render` — jamais depuis `@testing-library/react`
- `next/navigation`, `next/headers`, `next/image` sont mockés dans `vitest.setup.ts`

```tsx
// ✅
import { render, screen } from "@/test/utils/render"

// ❌
import { render } from "@testing-library/react"
```

### MSW (Mock Service Worker)

- Handlers globaux dans `src/test/mocks/handlers.ts`
- Surcharger ponctuellement dans un test avec `server.use(...)`
- Les surcharges sont automatiquement annulées après chaque test (`resetHandlers`)

```ts
import { server } from "@/test/mocks/server"
import { http, HttpResponse } from "msw"

server.use(http.get("/api/user", () => HttpResponse.json({ name: "Alice" })))
```

### Playwright

- Les specs sont dans `e2e/**/*.spec.ts` — **jamais dans `src/`**
- Chaque page a son **Page Object** dans `e2e/pages/`
- La méthode `goto()` doit **retourner** (pas `await`) `this.page.goto()` pour exposer la Response
- Priorité des locators : `getByRole` > `getByLabel` > `getByTestId`
- En CI : `playwright.config.ts` détecte `CI=true` et lance `pnpm build && pnpm start`

```ts
// ✅ retourne la Response
async goto() {
  return this.page.goto("/")
}

// ❌ avale la Response, retourne undefined
async goto() {
  await this.page.goto("/")
}
```

## Commits

Conventional Commits obligatoires (enforced par commitlint) :

```
feat:     nouvelle fonctionnalité
fix:      correction de bug
test:     ajout/modification de tests
ci:       pipeline CI/CD
build:    dépendances, scripts
docs:     documentation
refactor: refactorisation sans changement de comportement
chore:    tâches diverses
```

## Variables d'environnement

Ajouter dans `src/env.ts` (section `server` ou `client`), jamais lire `process.env` directement ailleurs.

```bash
NEXT_PUBLIC_APP_URL=http://localhost:3000  # obligatoire
NODE_ENV=development                        # automatique
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alexis-Gontier) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
