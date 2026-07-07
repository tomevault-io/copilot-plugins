## website2025

> Site Next.js de **Business Portugal** (création/implantation d'entreprise au Portugal), par Audrey Marques (Lovelyparallel, Lda). Audience : entrepreneurs **francophones**, dont ~90 % vivent **hors Portugal** (France ≫ Belgique, Suisse, Luxembourg, Monaco + Maghreb : Maroc, Tunisie, Algérie).

# CLAUDE.md — Business Portugal (`web/`)

Site Next.js de **Business Portugal** (création/implantation d'entreprise au Portugal), par Audrey Marques (Lovelyparallel, Lda). Audience : entrepreneurs **francophones**, dont ~90 % vivent **hors Portugal** (France ≫ Belgique, Suisse, Luxembourg, Monaco + Maghreb : Maroc, Tunisie, Algérie).

## Tech stack
Next.js 16 (App Router) · React 19 · TS strict · **Tailwind v4** (tokens CSS, zéro hex) · **next-intl** (`/fr` `/en`) · **Biome** (lint+format) · `zod` · `lucide-react` · cva/clsx/tailwind-merge. Gestionnaire de paquets : **pnpm**.

## Commandes
- `pnpm dev` · `pnpm build` · `pnpm start`
- `pnpm lint` (= `biome check .`) · `pnpm format` (= `biome format --write .`)
- Type-check : `pnpm exec tsc --noEmit`
- ⚠️ **Windows** : des builds répétés laissent des workers `node` orphelins → `VirtualAlloc failed` / `uv_spawn`. Remède : `Get-Process node | Stop-Process -Force` avant un build propre. En cours de dev, préférer `tsc --noEmit` (1 process) ; build complet seulement en fin de lot.

## Deux surfaces
- **`src/app/[locale]/*`** — site vitrine public, **bilingue fr/en (SSG)**, indexable. C'est le cœur SEO/GEO.
- **`src/app/(platform)/*`** (`admin`, `app`, `portal`, `launch`) — portail SaaS, **front-first mocké** (logique dans `src/platform/`, données `src/platform/mock/`). **Pas encore branché à Supabase** ; ne pas présumer de backend réel ici.

## Conventions NON négociables
- **Domaine & identité** : importer TOUT depuis `@/lib/site` (`SITE_URL`, `urlFor`, `languagesFor`, `ogLocaleFor`, IDs `@id`, coordonnées). **Jamais** coder le domaine/les coordonnées en dur. Domaine via `NEXT_PUBLIC_SITE_URL` (défaut `portugal-business.com`).
- **Design system** : suivre `web/DESIGN-SYSTEM.md` (contrat obligatoire). Classes sémantiques tokens uniquement (`bg-background`, `text-foreground`, `text-muted-foreground`, `text-accent`, `border-border`, `bg-card`, `bg-primary`), serif éditoriale (EB Garamond), classe `.eyebrow`, composant `<Reveal>` (`@/components/motion/reveal`), conteneur `site-frame`, boutons via `buttonVariants` (`@/components/ui/button`), `Link` de `@/i18n/navigation`. **Jamais** de hex, de variantes `dark:`, ni de `max-w-*` recopié en cadre.
- **i18n des nouvelles pages** : dictionnaire bilingue **local** par page (`const COPY = { fr, en }`), `setRequestLocale(locale)` en tête, `generateMetadata` avec `canonical` + `languages` + `ogLocaleFor`. Routes dynamiques → `generateStaticParams` couvrant fr+en.

## Architecture SEO/GEO (modèles à réutiliser)
- **Graphe d'entité** : `src/components/seo/structured-data.tsx` (JSON-LD global Organization/ProfessionalService/Person reliés par `@id`, `areaServed` multi-pays, `audience`, `contactPoint`). Les JSON-LD de page pointent vers ces `@id`.
- **Pages-pays (spokes)** : composant partagé `src/components/seo/spoke-page.tsx` (type `SpokeCopy`, impose les chunks GEO : convention+confiance, vrai risque, faits, parcours, FAQ, maillage, disclaimer). Pages minces sous `src/app/[locale]/depuis/<pays>/page.tsx`. Modèle = `depuis/belgique/page.tsx`. **Maghreb = branche séparée** (visa D2 + contrôle des changes, JAMAIS la logique CFC européenne).
- **Hub** : `src/app/[locale]/creer-societe-portugal-depuis-letranger/page.tsx`.
- **Blog** : articles en **module-par-fichier** dans `src/data/articles-francophone/<slug>.ts` (type `Article` depuis `src/data/article-types.ts`), agrégés par `articles-francophone/index.ts`, spreadés dans `src/data/articles.ts` (`articles = [...legacy, ...francophone]`). Le renderer `blog/[slug]/page.tsx` fournit déjà auteur + disclaimer + CTA ; un article ne porte que les données (+ `relatedLinks` optionnels pour le maillage). Modèle = comparatif `comparatifs/portugal-vs-france-espagne-dubai/page.tsx`.
- **Outils** : `src/lib/outils/` (`ifici-questions.ts` arbre client + verdict **server-side**, `cout-salarie.ts` calcul pur 2026, `tool-lead.ts`). Les outils ont une contextualisation pays **informative** (sélecteur + note) qui **n'altère pas** le verdict serveur.
- **Sitemap / robots** : `src/app/sitemap.ts` (routes statiques + articles auto via `BLOG_ROUTES`) et `src/app/robots.ts` (allow-all, aucun bot IA bloqué). Ajouter toute nouvelle route au sitemap.

## Règles éditoriales / YMYL (CRITIQUES, sujet fiscal sensible)
- **Audrey Marques = consultante** en implantation/création. **JAMAIS** comptable / expert-comptable / fiscaliste / avocate. Elle coordonne et met en relation (compta = Contabilista Certificado partenaire ; fiscalité fine = fiscaliste partenaire).
- **Terminologie portugaise exacte** : IRC (19 % / 15 % PME ≤ 50 000 €), IVA (TVA portugaise, 23 % — **jamais « TVA » pour le Portugal**), IRS, NIF/NIPC, ENI, Unipessoal Lda, Lda (capital **1 €/associé**), SA, IFICI (ex-RNH, **exclut les retraités**). Jamais de statuts FR (EURL/SARL/EI) pour le Portugal.
- **Interdits** : « 0 impôt », « garanti », « sans risque », « paradis fiscal » (promesse), « n°1 », faux avis/chiffres. Seule stat : « 75+ entrepreneurs accompagnés depuis 2025 ».
- **Faits sensibles** : datés (2026), niveau de confiance, « à valider par un fiscaliste ». **Ne jamais publier de date de convention non confirmée** (cas Maroc/Algérie). Décisions verrouillées : hreflang `fr` générique (jamais `fr-FR`), gTLD `.com`, pas de redirection géo-IP.
- Sources de vérité contenu : `../docs/BRIEF-CONTENU-CLIENTE.md` et `../docs/SEO-GEO/08-YMYL-EEAT-CONFORMITE.md`. Plan francophone exécuté : `../docs/SEO-GEO/12-PLAN-FRANCOPHONE.md`.

## Intégrations (env-driven, gated client)
`NEXT_PUBLIC_SITE_URL`, `NEXT_PUBLIC_CONTACT_EMAIL`, `NEXT_PUBLIC_PLAUSIBLE_DOMAIN` ; Brevo (email transactionnel), Supabase (tables `leads`/`tool_leads`), Calendly. Voir `.env.example`. La conversion (server actions) persiste en best-effort ; rien ne casse si une clé manque.

---
> Source: [portugal-business/website2025](https://github.com/portugal-business/website2025) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
