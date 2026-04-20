## srd

> Site vitrine professionnel pour **SRD Partners Sàrl** (fiduciaire, Genève).

# claude.md

## Projet
Site vitrine professionnel pour **SRD Partners Sàrl** (fiduciaire, Genève).

**Stack :** Next.js 15 App Router · TypeScript · Tailwind CSS 3
**Architecture :** propre, scalable, maintenable — zéro dépendance inutile.

---

## PRINCIPES OBLIGATOIRES

### 1. DRY (Don't Repeat Yourself)
- Aucune duplication de logique, de style ou de texte.
- Pattern répété → composant. Valeur répétée → token ou constante. Style répété → classe utilitaire.

### 2. Séparation des responsabilités
- UI séparée de la logique.
- Textes visibles → fichiers de traduction uniquement (`src/messages/*.json`).
- Données structurelles non-traduites (icônes, IDs, liens, coordonnées) → `src/lib/siteData.ts`.
- Tokens de design → `tailwind.config.ts` (couleurs, ombres, polices) + `src/app/globals.css` (classes utilitaires).

### 3. Clean Code
- Composants petits et spécialisés. Nommage explicite.
- Pas de code mort. Pas de `console.log`. Pas de style inline sauf si absolument nécessaire.

### 4. Scalabilité
- Ajout d'une nouvelle langue : créer `src/messages/<code>.json` + ajouter le code dans `src/lib/i18n.ts`.
- Ajout d'un service : ajouter l'entrée dans `siteData.ts` + les clés dans chaque fichier de traduction.
- Ajout d'une section : créer la page ou le bloc, réutiliser `<Section>`.

---

## SYSTÈME DE DESIGN

> Documentation complète : `design.md` (racine du projet)

### Tokens dans `tailwind.config.ts`
Les couleurs, ombres et polices sont définies dans le thème Tailwind — ne pas les hardcoder.

**Couleurs extraites du logo SRD Partners (v2) :**

| Token Tailwind      | Hex        | Usage                                    |
|---------------------|------------|------------------------------------------|
| `bg-cream`          | `#F8F9FC`  | Fond section principale (cool near-white)|
| `bg-stone`          | `#D2C9D6`  | Fond section alternée (cool light gray)  |
| `bg-navy`           | `#301B4A`  | Fond sombre — hero, footer               |
| `text-gold` / `bg-gold` | `#97144F` | Accent crimson (arrow logo) — CTA, overlines |
| `text-primary-500`  | `#301B4A`  | Violet logo SRD — titres, btn-outline    |
| `text-ink`          | `#1A0F2A`  | Texte principal dark                     |
| `text-muted`        | `#5A6080`  | Texte secondaire cool                    |
| `shadow-premium`    | multicouche| Cartes premium                           |
| `font-display`      | **Playfair Display** | Titres H1/H2, accent italique  |
| `font-body`         | **Manrope**          | Corps de texte, UI, forms      |

### Classes utilitaires dans `globals.css`
Définies dans `@layer components` / `@layer utilities` — toujours préférer ces classes :

| Classe             | Usage                              |
|--------------------|------------------------------------|
| `.container-main`  | Conteneur centré responsive        |
| `.section-label`   | Overline de section (uppercase)    |
| `.section-title`   | Titre H2 (Cormorant font-light, tracking -0.01em) |
| `.section-subtitle`| Sous-titre de section              |
| `.btn-primary`     | Bouton or (CTA principal)          |
| `.btn-outline`     | Bouton outline or                  |
| `.btn-outline-dark`| Bouton outline blanc (sur fond sombre) |
| `.card-base`       | Carte blanche avec hover           |
| `.animate-fade-up` | Animation d'entrée fadeUp          |
| `.hero-gradient`   | Dégradé navy/or du hero            |
| `.hero-grain`      | Texture grain sur le hero          |
| `.gold-line`       | Trait décoratif doré               |

**Interdit :** hardcoder couleurs, tailles, radius, ombres hors de ces systèmes.

---

## MULTILINGUE

### Architecture i18n custom (sans bibliothèque externe)

**Fichier central :** `src/lib/i18n.ts`
```ts
export type Locale = 'fr' | 'en' | 'pt'
export const locales: Locale[] = ['fr', 'en', 'pt']
export const defaultLocale: Locale = 'fr'

export function getMessages(locale: Locale): Messages { … }
export function isValidLocale(value: string): value is Locale { … }
```

**Usage dans les pages/layouts :**
```ts
const t = getMessages(locale)
// t.nav.home, t.hero.title, etc.
```

**Middleware :** `src/middleware.ts` — redirige `/` → `/<locale>` via cookie ou défaut `fr`.

### Fichiers de traduction
```
src/messages/
  fr.json   ← référence (toutes les clés)
  en.json   ← même structure
  pt.json   ← même structure
```
- Toutes les clés doivent être identiques dans les 3 fichiers.
- **Aucun texte visible en dur dans les composants.**

### Routing
```
/fr           → Page d'accueil (française)
/fr/services
/fr/a-propos
/fr/contact
/en/…
/pt/…
```

---

## COMPOSANTS TYPOGRAPHY

### Footer `slantFill`
Le `Footer` accepte une prop `slantFill?: string` (défaut `BG.stone`).
Doit être passée explicitement quand le `ContactBlock` précédent a `bg="cream"` :
```tsx
<Footer ... slantFill={BG.cream} />  {/* si ContactBlock bg="cream" */}
```

### `PremiumHeading` + `Accent` (`src/components/PremiumHeading.tsx`)
Système de titres premium : serif léger + fragment italique doré.

```tsx
import { PremiumHeading, Accent } from '@/components/PremiumHeading'

// Props : as (h1|h2|h3), size (hero|page|section), color (light|dark)
<PremiumHeading as="h1" size="page" color="light">
  {t.section.titleMain} <Accent>{t.section.titleAccent}</Accent>
</PremiumHeading>
```

- `size="hero"` → H1 plein écran (accueil)
- `size="page"` → H1 hero de sous-page (services, à-propos, contact)
- `size="section"` → H2 sections standard
- `color="light"` → fond navy (texte blanc)
- `color="dark"` → fond cream/stone (texte navy)
- `Accent` → italique + `text-gold` uniquement, **un seul fragment par titre**

Règle : ne jamais écrire `<span className="italic font-semibold text-gold">` directement dans une page.

### Clés de traduction pour les titres accentués
Les titres avec accent utilisent des clés `titleMain` + `titleAccent` dans les JSON.
Sections concernées : `features`, `services` (section home), heroes de pages (title1/title2 déjà existants).

---

## ASSETS STATIQUES

### Logo (`public/logo.png`)
- Fichier source : `logo.png` (racine du projet → copié dans `public/`)
- Format : PNG avec fond blanc, couleurs violet + cramoisi
- Utilisé dans : `Header.tsx` et `Footer.tsx` via `next/image`
- Sur fond navy : enveloppé dans un badge blanc arrondi `bg-white rounded-lg px-… py-…`
- **Ne jamais** afficher le logo directement sans ce badge sur fond sombre

```tsx
import Image from 'next/image'

// Header (h-7 sm:h-8) — compact
<div className="bg-white rounded-lg px-2.5 py-1.5 shrink-0">
  <Image src="/logo.png" alt={brandLegal} width={900} height={600} className="h-7 sm:h-8 w-auto" priority />
</div>

// Footer (h-10) — légèrement plus grand
<div className="bg-white rounded-lg px-3 py-2">
  <Image src="/logo.png" alt={t.brand.legal} width={900} height={600} className="h-10 w-auto" />
</div>
```

---

## STRUCTURE DU PROJET

```
public/
├── logo.png                     ← logo officiel (PNG, fond blanc)
└── images/
    └── check-list.png           ← illustration FAQ (violet, fond blanc transparent)
src/
├── app/
│   ├── globals.css              ← classes utilitaires, animations, base styles
│   ├── layout.tsx               ← root layout (minimal)
│   └── [locale]/
│       ├── layout.tsx           ← locale layout (lang, fonts, Header, SEO)
│       ├── page.tsx             ← page d'accueil
│       ├── services/page.tsx
│       ├── a-propos/page.tsx
│       └── contact/page.tsx
├── components/
│   ├── Header.tsx
│   ├── LanguageSwitcher.tsx
│   ├── Footer.tsx
│   ├── Section.tsx
│   ├── PremiumHeading.tsx       ← titres serif + Accent italique doré
│   ├── ScrollReveal.tsx         ← stagger GSAP à l'entrée dans le viewport
│   ├── ProcessSection.tsx       ← timeline horizontale pinned (GSAP ScrollTrigger)
│   ├── FaqSection.tsx           ← FAQ accordéon 2-col (image + accordéon)
│   ├── Card.tsx
│   ├── FeatureCard.tsx
│   ├── ServiceCard.tsx
│   ├── TestimonialCard.tsx
│   ├── TeamCard.tsx
│   ├── InfoCard.tsx
│   └── ContactBlock.tsx
├── lib/
│   ├── i18n.ts                  ← système i18n custom
│   ├── siteData.ts              ← données structurelles (icônes, IDs, contacts, BG)
│   ├── icons.tsx                ← composants SVG d'icônes
│   └── utils.ts                 ← cn() helper
├── messages/
│   ├── fr.json
│   ├── en.json
│   └── pt.json
└── middleware.ts                ← redirection locale
```

---

## COMPOSANT SECTION

`<Section>` gère les biseau diagonaux. Props :

```ts
bg?:        'cream' | 'stone' | 'navy' | 'white'  // fond
slant?:     'left' | 'right' | false               // direction du biseau supérieur
slantFill?: string                                  // couleur de la section PRÉCÉDENTE
id?:        string
noPadding?: boolean
```

- Le biseau est un SVG `aria-hidden="true"` — décoratif uniquement.
- `slantFill` doit correspondre à la couleur de fond de la section précédente.
- Ne jamais dupliquer le code du biseau hors de `Section.tsx`.

---

## COULEURS DE SECTION (`BG`)

```ts
import { BG } from '@/lib/siteData'
// BG.cream | BG.stone | BG.navy
```

Utiliser `BG` pour tous les `slantFill` — ne jamais hardcoder les hex dans les pages.

---

## DONNÉES STRUCTURELLES (`src/lib/siteData.ts`)

Centralise tout ce qui n'est PAS traduit :
- `services[]` — IDs et icônes des services
- `featureIcons[]` — icônes des points forts
- `contactInfo` — email, téléphone, adresses
- `socials[]` — liens réseaux sociaux

Les textes affichés de ces entités viennent des fichiers de traduction via leur `id`.

---

## SEO

Géré dans `src/app/[locale]/layout.tsx` via `generateMetadata()` :
- `html lang={locale}` dynamique
- `title` et `description` traduits via `getMessages()`
- `alternates.languages` → hreflang pour fr/en/pt + x-default
- `canonical` par locale
- Structure H1 unique par page, H2 pour les sections

---

## ACCESSIBILITÉ

- Focus visible : `*:focus-visible { outline: 2px solid gold }` (défini dans `globals.css`)
- Éléments décoratifs : `aria-hidden="true"`
- Inputs avec `<label>` associé
- Contrastes suffisants (gold sur blanc, blanc sur navy)
- Navigation clavier sur le menu mobile

---

## RÈGLES STRICTES

1. **Aucun texte visible en dur** dans les composants — tout vient de `getMessages()`.
2. **Aucune couleur/ombre/radius hardcodé** — utiliser les tokens Tailwind ou les classes utilitaires.
3. **Aucune duplication** de logique ou de style.
4. **Aucune dépendance npm inutile** — le projet n'a pas de `next-intl`, `clsx`, `framer-motion`, etc.
5. **TypeScript strict** — pas de `any`, typage explicite.
6. **Composants = UI seulement** — la logique/données sont dans `lib/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/linkio-web) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
