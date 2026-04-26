## vizly

> Builder de portfolios en ligne. Formulaire guidé → choix template → site live sur `pseudo.vizly.fr`.

# Vizly — Portfolio Builder SaaS

Builder de portfolios en ligne. Formulaire guidé → choix template → site live sur `pseudo.vizly.fr`.

## Stack

- **Next.js 15** (App Router) — frontend + backend
- **Supabase** — PostgreSQL, Auth (email + Google OAuth), Storage (images)
- **Stripe** — subscriptions (Starter 4.99€/Pro 9.99€) + one-shot templates premium (2.99€)
- **Resend** — emails transactionnels (noreply@vizly.fr)
- **Vercel** — hosting, wildcard `*.vizly.fr`
- **Tailwind CSS + shadcn/ui** — styling (toujours personnaliser shadcn)
- **Framer Motion** — animations
- **Playwright** — tests E2E

## MCPs

- **Supabase** : tables, migrations, RLS, storage, auth, SQL
- **Vercel** : deploy, domaines, env vars

## Conventions

- TypeScript strict : pas de `any`, `@ts-ignore`, `as unknown as`, `!`
- Server Components par défaut, `"use client"` uniquement si hooks/events/browser APIs
- Server Actions pour les mutations, Route Handlers pour les API publiques
- Zod pour toute validation (formulaires, API, webhooks)
- RLS sur 100% des tables Supabase
- Commits : `type(scope): description` (feat, fix, refactor, style, test)

# DESIGN-SYSTEM.md — Vizly

> Ce fichier est la source de vérité pour tout le design de Vizly.
> Claude Code DOIT lire ce fichier avant de toucher à n'importe quel composant UI.
> En cas de doute, la règle est : MOINS > PLUS.

---

## PHILOSOPHIE

Vizly doit ressembler à un SaaS conçu par une vraie équipe produit, pas à un side project vibe-coded.
Références visuelles : Apollo, Maze, Neon, Remote, Sprig.
ADN commun de ces apps : fond blanc, très peu de couleurs, beaucoup de whitespace,
inputs simples sans décoration, hiérarchie typographique claire, zéro effet superflu.

Mantra : **"Si tu hésites à ajouter quelque chose, ne l'ajoute pas."**

---

## COULEURS

```
--color-bg:              #FFFFFF        /* Fond principal, toujours blanc pur */
--color-bg-subtle:       #F9FAFB        /* Fond secondaire (sections alternées, sidebar) */
--color-bg-muted:        #F3F4F6        /* Fond inputs désactivés, hover léger */
--color-border:          #E5E7EB        /* Bordures par défaut — gris très léger */
--color-border-focus:    #D1D5DB        /* Bordures en focus — un cran plus foncé */

--color-text-primary:    #111827        /* Titres et texte principal — presque noir */
--color-text-secondary:  #6B7280        /* Texte secondaire, labels, descriptions */
--color-text-tertiary:   #9CA3AF        /* Placeholders, texte désactivé */

--color-accent:          #E8553D        /* Accent Vizly (le corail/rouge actuel) — UNIQUE couleur vive */
--color-accent-hover:    #D4442E        /* Hover sur accent */
--color-accent-light:    #FEF2F0        /* Background très léger pour badges/tags accent */

--color-success:         #059669        /* Vert validation — utilisé avec parcimonie */
--color-error:           #DC2626        /* Rouge erreur uniquement */
--color-warning:         #D97706        /* Orange warning uniquement */
```

### Règles couleurs
- L'accent corail (#E8553D) est utilisé UNIQUEMENT pour : le bouton CTA principal, le stepper actif, les liens d'action importants
- Maximum 1 élément accent visible à l'écran en même temps (hors stepper)
- Pas de gradients. Jamais. Nulle part.
- Pas de couleurs de fond colorées sur les sections (pas de bg-blue-50, bg-purple-50, etc.)
- Les icônes sont en --color-text-secondary (#6B7280), jamais en couleur

---

## TYPOGRAPHIE

```
--font-family:           'DM Sans', system-ui, sans-serif
```

DM Sans pour tout. Pas de deuxième police. Pas de serif. Pas de monospace sauf code.

### Échelle typographique (stricte, pas d'autres tailles)

| Token             | Taille  | Poids | Line-height | Usage                           |
|--------------------|---------|-------|-------------|---------------------------------|
| --text-page-title  | 24px    | 600   | 32px        | Titre de page ("Mes projets")   |
| --text-section     | 18px    | 600   | 28px        | Titre de section dans un form   |
| --text-subsection  | 14px    | 600   | 20px        | Sous-titre, label de catégorie  |
| --text-body        | 14px    | 400   | 20px        | Corps de texte, labels de champ |
| --text-small       | 13px    | 400   | 18px        | Descriptions, helper text       |
| --text-caption     | 12px    | 500   | 16px        | Compteurs (4/30), badges, meta  |

### Règles typo
- Pas de texte en bold (700) sauf dans les boutons
- Pas de texte en uppercase sauf les badges de statut (PRO, BROUILLON)
- Pas d'italique
- Pas de text-decoration sauf :hover sur les liens
- Les labels de champ sont en --text-body (14px/400), couleur --color-text-secondary
- Les titres de section n'ont PAS d'icône devant eux (retire les icônes décoratives)

---

## SPACING

Grille de 4px. Tous les espacements sont des multiples de 4.

```
--space-xs:    4px
--space-sm:    8px
--space-md:    12px
--space-base:  16px
--space-lg:    24px
--space-xl:    32px
--space-2xl:   48px
--space-3xl:   64px
```

### Règles spacing
- Padding intérieur d'une page : 32px horizontal, 32px vertical
- Gap entre sections de formulaire : 32px (pas de cards — juste du whitespace + un border-bottom)
- Gap entre champs dans une section : 16px
- Padding intérieur d'une card : 20px
- Marge entre le label et l'input : 6px
- Marge entre l'input et le helper text : 4px

---

## COMPOSANTS

### Inputs (texte, textarea, select)

```css
input, textarea, select {
  height: 40px;                          /* 44px pour les textareas (auto-height) */
  padding: 8px 12px;
  border: 1px solid var(--color-border); /* #E5E7EB */
  border-radius: 8px;
  font-size: 14px;
  color: var(--color-text-primary);
  background: var(--color-bg);           /* Blanc pur */
  transition: border-color 0.15s ease;
}

input:focus {
  border-color: var(--color-border-focus); /* #D1D5DB — PAS de couleur vive */
  outline: none;
  box-shadow: 0 0 0 3px rgba(0, 0, 0, 0.04); /* Ombre quasi invisible */
}

input::placeholder {
  color: var(--color-text-tertiary);     /* #9CA3AF */
}
```

#### CE QUI EST INTERDIT sur les inputs :
- ❌ Icône à gauche dans l'input (pas d'icône LinkedIn dans le champ LinkedIn)
- ❌ Border-color en accent/rouge au focus
- ❌ Background coloré au focus
- ❌ Border-radius > 8px
- ❌ Box-shadow visible au repos
- ❌ Height > 44px (sauf textarea)

### Boutons

**Bouton primaire (CTA) :**
```css
.btn-primary {
  height: 40px;
  padding: 0 20px;
  background: var(--color-accent);       /* #E8553D */
  color: #FFFFFF;
  border: none;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: background 0.15s ease;
}
.btn-primary:hover {
  background: var(--color-accent-hover); /* #D4442E */
}
```

**Bouton secondaire :**
```css
.btn-secondary {
  height: 40px;
  padding: 0 20px;
  background: var(--color-bg);
  color: var(--color-text-primary);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  font-size: 14px;
  font-weight: 500;
}
```

**Bouton texte (back, annuler) :**
```css
.btn-text {
  background: none;
  border: none;
  color: var(--color-text-secondary);
  font-size: 14px;
  font-weight: 500;
  padding: 0;
}
```

#### Règles boutons
- Maximum 1 bouton primaire visible par écran
- "Précédent" = bouton texte (pas un bouton outline), positionné à GAUCHE du bouton principal
- Navigation bas de page : aligner Précédent et Suivant à DROITE, côte à côte
- Pas d'icône dans les boutons sauf + (ajouter) et → (suivant)
- Pas de boutons en full-width sauf sur mobile

### Cards

```css
.card {
  background: var(--color-bg);
  border: 1px solid var(--color-border);
  border-radius: 12px;                  /* 12px max pour les cards */
  padding: 20px;
  box-shadow: none;                     /* PAS d'ombre au repos */
}
.card:hover {
  border-color: var(--color-border-focus);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.04); /* Ombre ultra subtile au hover */
}
```

#### CE QUI EST INTERDIT sur les cards :
- ❌ box-shadow au repos (shadow-sm, shadow-md, etc.)
- ❌ border-radius > 12px
- ❌ Background coloré (bg-red-50, bg-blue-50, etc.)
- ❌ Bordure colorée (border-left accent, etc.)
- ❌ Header de card avec fond coloré

### Tags / Chips (compétences, technologies)

```css
.tag {
  display: inline-flex;
  align-items: center;
  height: 28px;
  padding: 0 10px;
  background: var(--color-bg-muted);    /* #F3F4F6 */
  color: var(--color-text-primary);
  border: none;                         /* PAS de border */
  border-radius: 6px;
  font-size: 13px;
  font-weight: 400;
  gap: 6px;
}
```

- Le bouton × pour supprimer un tag : couleur --color-text-tertiary, 14px
- Pas de border sur les tags
- Pas de couleurs différentes par tag

### Badges de statut

```css
.badge {
  display: inline-flex;
  height: 22px;
  padding: 0 8px;
  border-radius: 4px;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.02em;
  text-transform: uppercase;
}
.badge-pro     { background: var(--color-accent-light); color: var(--color-accent); }
.badge-draft   { background: #F3F4F6; color: #6B7280; }
.badge-live    { background: #ECFDF5; color: #059669; }
```

### Upload zone (drag & drop images)

```css
.upload-zone {
  border: 1.5px dashed var(--color-border);  /* Gris, PAS coloré */
  border-radius: 12px;
  padding: 32px;
  text-align: center;
  background: var(--color-bg-subtle);        /* #F9FAFB */
  cursor: pointer;
  transition: border-color 0.15s ease;
}
.upload-zone:hover {
  border-color: var(--color-border-focus);
}
```

- ❌ INTERDIT : bordure en rouge/rose/accent sur la zone d'upload
- ❌ INTERDIT : background coloré sur la zone d'upload
- L'icône upload : 24px, couleur --color-text-tertiary
- Texte : "Glisse ou clique pour ajouter" en --color-text-secondary, 14px
- Sous-texte : "JPG, PNG, WebP — max 5 images" en --color-text-tertiary, 13px

---

## LAYOUT

### Sidebar (navigation principale)

**Option recommandée : sidebar fixe, toujours visible, fine**

```
Largeur : 220px (desktop), rétractable à 56px (icônes seules)
Background : var(--color-bg-subtle) ou var(--color-bg)
Border-right : 1px solid var(--color-border)
```

- Le toggle collapse/expand se fait via un bouton chevron, PAS au hover
- Items de nav : height 36px, padding-left 12px, border-radius 6px
- Item actif : background var(--color-bg-muted), font-weight 500
- Item hover : background var(--color-bg-muted)
- Icônes de nav : 18px, stroke-width 1.5, couleur --color-text-secondary
- Icône de l'item actif : couleur --color-text-primary
- Pas de couleur accent sur les items de nav (pas de fond rouge/corail)

Référence : la sidebar de Neon, Maze, Sprig — sobre, texte noir, fond neutre

### Stepper (Profil → Projets → Contenu → Design → Publier)

```
Style : ligne horizontale avec dots/numéros
Couleur étape complétée : var(--color-accent) pour le dot, ligne en --color-accent
Couleur étape active : dot rempli en --color-accent, texte en --color-text-primary
Couleur étape future : dot vide border --color-border, texte en --color-text-tertiary
Ligne entre les étapes : 2px de haut, connecte les dots
```

- Pas de background-circle coloré autour de l'icône d'étape
- Pas d'icône custom par étape — juste un numéro ou un dot
- Labels sous les dots en --text-caption (12px/500)

### Split panel (éditeur + preview)

```
Panel gauche (éditeur) : flex-1, max-width 520px, padding 32px, scroll-y
Panel droit (preview)  : flex-1, background var(--color-bg-subtle), border-left 1px
Séparation             : 1px solid var(--color-border), pas de gap
```

### Sections de formulaire

**NE PAS wrapper chaque section dans une card.**

Utiliser un pattern plus léger :

```
Section title (18px/600)
Description (13px/400, --color-text-secondary)
                                        ← 16px gap
Champ 1
                                        ← 16px gap
Champ 2
                                        ← 32px gap
─────────────────────────────           ← border-bottom 1px solid var(--color-border)
                                        ← 32px gap
Section title suivante
```

Si des cards sont absolument nécessaires (ex: card de projet dans le dashboard),
elles suivent le style .card défini plus haut — pas d'ombre, border 1px, radius 12px.

---

## ANIMATIONS & TRANSITIONS

```css
/* La seule transition autorisée */
transition: [property] 0.15s ease;
```

### Ce qui a une transition :
- border-color au focus/hover
- background-color au hover (boutons, items de nav)
- opacity pour les apparitions (0 → 1)
- transform: translateY pour les entrées de page (0, 4px → 0, 0) — subtil

### CE QUI EST INTERDIT :
- ❌ Animations au scroll (pas de fade-in-up au scroll)
- ❌ Transitions > 0.2s
- ❌ Scale au hover (pas de transform: scale(1.02))
- ❌ Bounce, elastic, spring animations
- ❌ Skeleton loaders animés avec shimmer (utiliser un simple spinner ou texte "Chargement...")
- ❌ Hover effects qui changent la taille d'un élément (shift le layout)

---

## ICÔNES

Provider : Lucide React (déjà utilisé dans le projet)

```
Taille par défaut : 18px
Stroke-width : 1.5
Couleur par défaut : var(--color-text-secondary)
```

### Règles icônes
- Les icônes ne sont JAMAIS en couleur accent sauf dans les boutons CTA
- Les icônes ne sont JAMAIS dans un cercle/carré coloré (pas de bg-red-100 + icône rouge)
- Les icônes devant les labels de section sont SUPPRIMÉES — texte seul suffit
- Les icônes dans les inputs sont SUPPRIMÉES (pas d'icône LinkedIn dans le champ URL)
- Exception : icônes dans la sidebar de navigation (nécessaires)

---

## RESPONSIVE

```
Mobile  : < 768px  → sidebar cachée, hamburger menu, formulaire full-width
Tablet  : 768-1024px → sidebar rétractée (56px icônes), preview cachée
Desktop : > 1024px → layout complet sidebar + éditeur + preview
```

- Sur mobile, le split panel disparaît — le preview est accessible via un bouton "Aperçu"
- Les boutons Précédent/Suivant deviennent sticky en bas sur mobile
- Padding de page : 16px sur mobile, 24px sur tablet, 32px sur desktop

---

## ANTI-PATTERNS — LISTE EXHAUSTIVE DES CHOSES À NE JAMAIS FAIRE

1. ❌ Gradients (sur quoi que ce soit)
2. ❌ Box-shadow au repos (shadow-sm, shadow-md, shadow-lg, shadow-xl)
3. ❌ Border-radius > 12px (pas de rounded-2xl, rounded-3xl, rounded-full sur des cards)
4. ❌ Background coloré sur les sections (bg-red-50, bg-blue-50, bg-gradient-to-r)
5. ❌ Icônes dans des cercles/carrés colorés
6. ❌ Plus de 2 font-weight visibles par page (400 et 600, c'est tout)
7. ❌ Texte centré sur les formulaires (toujours aligné à gauche)
8. ❌ Bordures colorées (border-l-4 border-accent, etc.)
9. ❌ Hover effects qui scale ou lift les éléments
10. ❌ Uppercase sur autre chose que les badges de statut
11. ❌ Emojis dans l'interface (pas de 👋 dans "Bienvenue")
12. ❌ Placeholder text qui fait "fun" — rester factuel
13. ❌ Dividers avec icônes ou texte au milieu
14. ❌ Tabs avec background coloré sur l'onglet actif — utiliser un underline 2px
15. ❌ Loading skeletons shimmer — utiliser un spinner simple
16. ❌ Toast notifications colorées full-width — petits toasts en bas à droite, border, sobre
17. ❌ Modales avec overlay très opaque — overlay à opacity 0.3 max
18. ❌ Bordure dashed en couleur (surtout pas en accent/rouge pour les uploads)
19. ❌ Multiple couleurs d'accent — UNE SEULE : le corail #E8553D

---

## CHECKLIST AVANT CHAQUE COMMIT UI

Avant de valider un changement UI, Claude Code vérifie :

- [ ] Aucun gradient visible
- [ ] Aucun box-shadow au repos
- [ ] Aucun border-radius > 12px
- [ ] Aucune icône dans un cercle coloré
- [ ] Maximum 1 bouton primaire (accent) visible à l'écran
- [ ] Les inputs n'ont pas d'icône intégrée
- [ ] Les labels sont en 14px/400 gris, pas en bold
- [ ] Les sections de form sont séparées par du whitespace + border-bottom, pas des cards
- [ ] La zone d'upload est en border dashed GRIS, pas en couleur
- [ ] Le texte est aligné à gauche (pas centré) dans les formulaires
- [ ] Les couleurs utilisées sont UNIQUEMENT celles définies dans ce fichier

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomlefb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
