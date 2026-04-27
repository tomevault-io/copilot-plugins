## kmee-website

> - **Astro 6** + **Tailwind CSS 4** + **React 19** + **TypeScript**

# KMEE Website — Convenções para Vibecoding

## Stack
- **Astro 6** + **Tailwind CSS 4** + **React 19** + **TypeScript**
- Output: **Static** por padrão (Astro 6), SSR com `export const prerender = false` em páginas individuais
- Deploy: Vercel

## Cores
- Brand: `brand-500: #2e6885` (azul/teal) — cor primária
- Accent: `accent-400: #41c452` (verde) — CTAs e destaques
- Tokens definidos em `src/styles/globals.css`

## Estrutura
- `src/components/landing/` — Seções da homepage
- `src/components/layout/` — Navbar + Footer
- `src/components/solutions/` — Componentes de páginas de soluções
- `src/components/ui/` — Primitivos: Button, Card, SectionHeader
- `src/components/islands/` — React islands (client:load)
- `src/data/` — Dados estruturados (indústrias)
- `src/i18n/ui.ts` — Strings i18n, usar `t("pt", "key")`
- `src/lib/odoo.ts` — Cliente JSON-RPC para Odoo
- `src/layouts/` — Base.astro e BlogPost.astro
- `src/content.config.ts` — Schema do blog com Content Collections (deve estar em src/ no Astro 6)

## Padrões
- Componentes Astro com Tailwind puro, **sem shadcn**
- React apenas para interatividade (formulários) — usar `client:load`
- Páginas SSR: carreiras, cursos — sempre tratar erro do Odoo com fallback
- i18n: usar `t(lang, "key")` de `@/i18n/ui`
- SEO: title e description em todas as páginas via Base.astro props

## Comandos
```bash
npm run dev     # Dev server
npm run build   # Build para produção
npm run preview # Preview do build
```

## Odoo
- Variáveis em `.env` (ODOO_URL, ODOO_DB, ODOO_USER, ODOO_PASSWORD)
- Usar `getOdooClient()` de `@/lib/odoo.ts`
- Sempre cache SSR: `Astro.response.headers.set('Cache-Control', 's-maxage=300')`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kmee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
