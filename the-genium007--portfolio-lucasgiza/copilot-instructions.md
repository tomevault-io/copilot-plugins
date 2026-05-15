## portfolio-lucasgiza

> Portfolio personnel Full-Stack Developer. Site statique, pas d'API backend.

# Portfolio Lucas GIZA — lucasgiza.com

Portfolio personnel Full-Stack Developer. Site statique, pas d'API backend.

## Stack

- **Framework** : Next.js 16 (App Router) — JavaScript/JSX (migration TypeScript possible en 2026)
- **React** : 19.2 avec Server Components
- **Styling** : Tailwind CSS v4 + design tokens CSS (`styles/tokens.css`)
- **Déploiement** : Docker standalone → Dokploy

## Commandes

```bash
npm run dev        # Dev server (port 3005)
npm run build      # Build production
npm run lint       # ESLint (flat config)
npm run format     # Prettier (tous les fichiers)
```

Toujours vérifier `npm run build` avant de considérer une tâche terminée.

## Architecture

```
app/            → Pages (App Router, pas de pages/)
components/     → Composants réutilisables (layout/, ui/, navigation/, projects/, icons/)
sections/       → Sections de page (home/, experience/, projects/, skills/)
data/           → Données statiques (projects.js, experience.js, skills.js, etc.)
lib/            → Hooks et utilitaires (cn.js, useScrollSpy.js, metadata.js, fonts.js)
styles/         → CSS global + tokens + components + utilities
public/         → Assets statiques (favicon/, og-image.png)
```

## Conventions de code

- **Langage** : JavaScript/JSX uniquement (pas de TypeScript pour l'instant)
- **Imports** : Utiliser l'alias `@/` (ex: `import { cn } from '@/lib/cn'`)
- **Composants** : Fichiers `.jsx`, nommage kebab-case (ex: `highlight-project-card.jsx`)
- **Exports** : `export default` pour les composants, named exports pour les utilitaires
- **Styling** : Classes Tailwind + `cn()` (clsx + tailwind-merge). Design tokens via CSS variables dans `tokens.css`
- **Formatter** : Prettier (single quotes, semi, tabWidth 2, printWidth 100, trailing comma es5)
- **Linter** : ESLint 9 flat config, extends `next/core-web-vitals`

## Conventions de commits (Conventional Commits)

Format : `type(scope): description`

Types : `feat`, `fix`, `style`, `refactor`, `docs`, `chore`, `perf`
Scopes courants : `ui`, `layout`, `data`, `seo`, `deploy`, `config`

Exemples :
- `feat(ui): add dark mode toggle`
- `fix(layout): correct sidebar overflow on mobile`
- `chore(config): update eslint rules`

## Workflow Git

- **main** : branche de production, toujours stable
- **development** : branche de développement actif
- Travailler sur `development`, merger dans `main` quand stable

## Données du site

Les données sont dans `data/` sous forme d'objets JS exportés. Pour modifier le contenu du portfolio :
- Projets : `data/projects.js` et `data/personal-projects.js`
- Expérience : `data/experience.js`
- Compétences : `data/skills.js`
- Navigation : `data/navigation.js`
- Métadonnées : `data/site.js`

## Responsive

- **Desktop** : Sidebar navigation (`components/layout/sidebar.jsx`)
- **Mobile** : Header + floating nav (`components/navigation/mobile-floating-nav.jsx`)
- Breakpoint clé : Tailwind `md:` pour le switch desktop/mobile

## Points d'attention

- Le build est en mode `standalone` (optimisé Docker) — ne pas changer sans raison
- Images : WebP + AVIF activés, cache 1 an — toujours utiliser `next/image`
- Les `console.log` sont automatiquement supprimés en production (compiler Next.js)
- Analytics : Umami intégré dans `app/layout.js`
- SEO : Sitemap dynamique (`app/sitemap.js`), robots.txt, Open Graph configurés
- Fonts : Inter (sans) + JetBrains Mono (mono) via Google Fonts

---
> Source: [The-Genium007/portfolio-lucasgiza](https://github.com/The-Genium007/portfolio-lucasgiza) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
