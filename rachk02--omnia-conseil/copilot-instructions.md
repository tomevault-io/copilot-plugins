## omnia-conseil

> **Projet :** Site web institutionnel pour "Omnia Conseil".

# CONTEXTE PROJET : OMNIA CONSEIL

## 1. Identité & Mission

**Projet :** Site web institutionnel pour "Omnia Conseil".
**Modèle d'inspiration :** [Fiserv.com](https://www.fiserv.com/) (Structure corporate, design "clean", navigation riche, confiance et robustesse).
**Activités :**

1.  **Conseil Financier :** Gestion, fiscalité, comptabilité, audit.
2.  **Infrastructure Numérique :** Audit IT, matériel (bureautique/réseau), déploiement logiciel, maintenance.
3.  **Data & IA :** Business Intelligence, Data Insights, Conception d'agents IA sur mesure.

**Langues :** français (défaut), anglais, chinois (Simplifié), arabe (RTL).

## 2. Stack Technique (Strict)

-   **Framework :** Next.js 16+ (App Router).
-   **Langage :** TypeScript.
-   **Styling :** Tailwind CSS.
-   **i18n :** `next-intl` (Gestion du routage et des traductions JSON).
-   **Icônes :** `lucide-react`.
-   **Polices :** Inter (défaut), Noto Sans Arabic (pour l'arabe, RTL), Noto Sans SC (pour le chinois).
-   **IDE :** WebStorm.
-   **Package Manager :** npm.
-   **Librairies additionnelles :** `clsx`, `tailwind-merge`.

## 3. Architecture & Structure des Fichiers

L'architecture suit les conventions **Next.js App Router** avec internationalisation.

```
omnia-conseil/
├── .gitignore
├── messages/                  # Fichiers de traduction (fr.json, en.json, ar.json, zh.json)
├── middleware.ts              # Middleware de redirection (à la racine du projet !)
├── next.config.ts             # Config avec plugin next-intl
├── tailwind.config.ts         # Config design system
├── package.json
├── package-lock.json
├── postcss.config.mjs
├── tsconfig.json
├── public/                    # Fichiers statiques (images, favicon, etc.)
├── src/
│   ├── app/                   # Application racine
│   │   ├── [locale]/          # Dossier dynamique OBLIGATOIRE pour i18n
│   │   │   ├── layout.tsx     # LocaleLayout (Gestion dir="rtl/ltr", fonts, et i18n)
│   │   │   ├── page.tsx       # Page d'accueil
│   │   │   ├── about/         # Page à propos
│   │   │   │   └── page.tsx
│   │   │   ├── contact/       # Page de contact
│   │   │   │   └── page.tsx
│   │   │   ├── insights/      # Page insights/blog
│   │   │   │   └── page.tsx
│   │   │   ├── not-found.tsx  # Page d'erreur 404 avec design Nano Banana
│   │   │   └── solutions/     # Sous-dossier pour les solutions
│   │   │       ├── finance/   # Solution Finance
│   │   │       │   └── page.tsx
│   │   │       ├── it-infra/  # Solution Infrastructure IT
│   │   │       │   └── page.tsx
│   │   │       └── data-ai/   # Solution Data & IA
│   │   │           └── page.tsx
│   │   ├── favicon.ico
│   │   ├── globals.css        # Styles CSS globaux (y compris pour 404)
│   │   └── layout.tsx         # Layout racine (RootLayout)
│   ├── components/            # Composants réutilisables
│   │   ├── layout/            # Composants de mise en page
│   │   │   ├── Footer.tsx     # Pied de page global
│   │   │   └── Navbar.tsx     # Barre de navigation
│   │   ├── sections/          # Sections spécifiques aux pages
│   │   │   ├── Hero.tsx       # Section hero de la page d'accueil
│   │   │   ├── ServicesGrid.tsx # Grille des services
│   │   │   ├── Stats.tsx      # Section statistiques
│   │   │   └── Values.tsx     # Section valeurs
│   │   └── icons/             # Composants d'icônes
│   │       └── XIcon.tsx      # Icône X (remplace Twitter)
│   ├── i18n/                  # Configuration internationale
│   │   ├── request.ts         # Configuration serveur i18n
│   │   └── routing.ts         # Configuration du routage i18n
│   ├── navigation.ts          # Helpers (Link, useRouter) via next-intl
│   ├── types/                 # Définitions de types TypeScript
│   └── lib/                   # Fonctions utilitaires
└── README.md
```

### Structure Détailée

#### 1. **Dossier `messages/`**
Contient les fichiers de traduction pour chaque langue supportée :
- `en.json` - Anglais
- `ar.json` - Arabe (RTL)
- `fr.json` - Français
- `zh.json` - Chinois

#### 2. **Dossier `src/app/[locale]/`**
C'est le cœur de l'internationalisation. Chaque page dans ce dossier est dynamique et générée pour chaque langue.

##### Pages disponibles :
- `page.tsx` - Page d'accueil
- `about/page.tsx` - Page "À propos"
- `contact/page.tsx` - Page de contact
- `insights/page.tsx` - Page insights/blog
- `solutions/` - Dossier des pages de services
    - `finance/page.tsx` - Solution financière
    - `it-infra/page.tsx` - Solution infrastructure IT
    - `data-ai/page.tsx` - Solution Data & IA
- `not-found.tsx` - Page d'erreur 404 avec design "Nano Banana"

#### 3. **Composants réutilisables (`src/components/`)**
- `layout/` : Composants de structure globale
    - `Footer.tsx` : Pied de page avec liens sociaux et rapides
    - `Navbar.tsx` : Barre de navigation avec menu et sélecteur de langue
- `sections/` : Sections spécifiques à certaines pages
    - `Hero.tsx` : Section d'en-tête avec appel à l'action
    - `ServicesGrid.tsx` : Grille des services proposés
    - `Stats.tsx` : Section avec statistiques clés
    - `Values.tsx` : Section présentant les valeurs de l'entreprise
- `icons/` : Composants d'icônes SVG personnalisées
    - `XIcon.tsx` : Icône X (anciennement Twitter)

#### 4. **Internationalisation (`src/i18n/`)**
- `request.ts` : Configuration du serveur pour le système de traduction
- `routing.ts` : Configuration du routage internationalisé

#### 5. **Page d'erreur 404 personnalisée**
La page `not-found.tsx` implémente un design moderne appelé "Nano Banana" avec :
- Effet glassmorphism
- Arrière-plan animé (lava lamp tech)
- Grand chiffre 404 en filigrane
- Icônes modernes et boutons stylisés
- Entièrement localisable (traduit dans toutes les langues)

Cette structure permet une modularité maximale, un bon découplage des responsabilités et une gestion efficace de l'internationalisation.

## 4. Règles de Développement (Critique)

### A. Next.js 16 & Asynchronisme

**Règle absolue :** Les `params` et `searchParams` sont désormais des **Promesses**.
Ne jamais accéder à `params.locale` directement.

```typescript
// ❌ MAUVAIS
export default function Page({ params }) { console.log(params.locale); }

// ✅ BON
export default async function Page({ params }: { params: Promise<{ locale: string }> }) {
  const { locale } = await params;
  // ...
}
```

### B. Internationalisation & RTL (Arabe)

1.  **Traductions :** Aucun texte en dur dans le JSX. Utiliser :
    -   `const t = useTranslations('SectionName');` (Client Components)
    -   `const t = await getTranslations('SectionName');` (Server Components)
2.  **Support RTL (Right-to-Left) :** Pour l'Arabe, le layout s'inverse.
    -   Ne jamais utiliser `text-left` ou `text-right` → Utiliser **`text-start`** et **`text-end`**.
    -   Ne jamais utiliser `pl-4` ou `pr-4` → Utiliser **`ps-4`** (padding-start) et **`pe-4`** (padding-end).
    -   Ne jamais utiliser `ml-4` ou `mr-4` → Utiliser **`ms-4`** (margin-start) et **`me-4`** (margin-end).
3.  **Polices spécifiques :** Pour le support RTL (arabe) et les scripts non latins (chinois), utiliser les polices spécifiques :
    -   Arabe : `Noto_Sans_Arabic` avec la variable CSS `--font-arabic`
    -   Chinois : `Noto_Sans_SC` avec la variable CSS `--font-chinese`
    -   Autres langues : `Inter` (par défaut)

### C. Composants & Images

-   Utiliser `<Image />` de `next/image` pour l'optimisation.
-   Utiliser `'use client'` uniquement si nécessaire (interactivité, hooks).
-   Utiliser `clsx` et `tailwind-merge` pour concaténer les classes CSS si besoin.

## 5. Configuration du Routage i18n

Le routage internationalisé est configuré dans `src/i18n/routing.ts` :

```typescript
import {defineRouting} from 'next-intl/routing';

export const routing = defineRouting({
    // La liste de toutes les langues supportées
    locales: ['fr', 'en', 'ar', 'zh'],

    // La langue par défaut (qui n'aura pas de préfixe)
    defaultLocale: 'fr',

    // 'as-needed' signifie : "Ne mets le préfixe que si ce n'est pas la langue par défaut"
    localePrefix: 'as-needed'
});
```

Le middleware est configuré dans `middleware.ts` pour rediriger selon la configuration de routage.

## 6. Design System & Mapping (Inspiration Fiserv)

Le design doit refléter le sérieux de Fiserv mais adapté à Omnia.

### Mapping des Pages

| URL Omnia                 | Contenu & Inspiration Fiserv                                 |
| :------------------------ | :----------------------------------------------------------- |
| **`/` (Home)**            | Hero immersif, "Bento grid" des 3 pôles, Preuve sociale.     |
| **`/solutions/finance`**  | Inspiré de *Risk & Compliance*. Focus: Fiscalité, Audit, Sérénité. |
| **`/solutions/it-infra`** | Inspiré de *Hardware Procurement*. Focus: Matériel, Réseau, Solidité technique. |
| **`/solutions/data-ai`**  | Inspiré de *Data Analytics*. Focus: Innovation, Dashboards, Agents IA. |
| **`/insights`**           | Inspiré de *Insights & Case Studies*. Blog technique et réalisations clients. |
| **`/about`**              | Inspiré de *Executive Leadership*. Présentation des experts. |
| **`/contact`**            | Formulaire dynamique avec sélecteur de pôle.                 |

### Palette (Suggestion)

-   **Primaire :** Orange dynamique (type Fiserv) ou Bleu Corporate profond.
-   **Secondaire :** Gris ardoise (Slate-900) pour le texte.
-   **Accent :** Couleur distincte pour les boutons CTA.

## 7. Instructions pour Gemini CLI (Assistant Design & UX)

**Rôle principal :** Design system, UX/UI, et composants innovants

### 7.1. Spécialités créatives
- **Design System moderne** : Glassmorphism, neumorphism, gradients animés
- **Animations Framer Motion** : Micro-interactions, transitions complexes, scroll-based animations
- **UX/UI innovant** : Composants interactifs, expériences immersives
- **Responsive Design** : Mobile-first, adaptatif, accessible
- **Support RTL/i18n** : Design adapté pour toutes les langues et directions

### 7.2. Directives de design
1. **Créer des composants visuellement attractifs** avec des animations fluides
2. **Utiliser Framer Motion** pour des interactions sophistiquées et performantes
3. **Respecter les contraintes RTL** dans tous les designs (text-start/end, ps/pe, ms/me)
4. **Maintenir la cohérence** avec le design system corporate inspiré de Fiserv
5. **Optimiser l'expérience utilisateur** avec des micro-interactions pertinentes

### 7.3. Exemples d'utilisation
```
gemini "créer un composant hero avec animation glassmorphism"
gemini "designer une section de statistiques avec animations scroll-based"
gemini "créer un composant interactif avec effet de parallaxe"
gemini "designer une navigation mobile avec animations fluides"
```

### 7.4. Contraintes créatives
- **Palette corporate** : Orange dynamique ou Bleu Corporate profond, Gris ardoise
- **Inspiration Fiserv** : Design "clean", navigation riche, confiance et robustesse
- **Support multilingue** : Adapter tous les designs pour l'arabe (RTL) et les autres langues
- **Performance** : Utiliser Framer Motion de manière optimisée
- **Accessibilité** : Maintenir les standards WCAG dans tous les designs

### 7.5. Composants à créer/améliorer
- Sections hero avec animations immersives
- Grilles de services avec interactions hover
- Composants de témoignages avec carrousels animés
- Calculators interactifs (ROI, etc.)
- Cartes du monde avec animations
- Timelines animées pour l'histoire de l'entreprise

## 8. Gemini CLI - Analyse Massive de Codebase

### 8.1. Syntaxe d'inclusion de fichiers avec @

Utilise la syntaxe `@` pour inclure des fichiers et répertoires dans les prompts Gemini. Les chemins sont relatifs au répertoire courant où vous exécutez la commande gemini.

#### Exemples d'utilisation :

**Analyse d'un seul fichier :**
```bash
gemini -p "@src/components/Hero.tsx Expliquer le purpose et la structure de ce fichier"
```

**Analyse de plusieurs fichiers :**
```bash
gemini -p "@package.json @src/app/layout.tsx Analyser les dépendances et la structure"
```

**Analyse d'un répertoire entier :**
```bash
gemini -p "@src/components/ Résumer l'architecture des composants"
```

**Analyse de tout le projet :**
```bash
gemini -p "@./ Donner une vue d'ensemble de tout le projet"
```

**Ou utiliser --all_files :**
```bash
gemini --all_files -p "Analyser la structure du projet et les dépendances"
```

### 8.2. Exemples de vérification d'implémentation

**Vérifier si une fonctionnalité est implémentée :**
```bash
gemini -p "@src/ @lib/ Le mode sombre est-il implémenté ? Montrer les fichiers pertinents"
```

**Vérifier l'implémentation i18n/RTL :**
```bash
gemini -p "@src/ @i18n/ Le support RTL est-il correctement implémenté ? Lister tous les composants concernés"
```

**Vérifier les patterns React :**
```bash
gemini -p "@src/ Y a-t-il des hooks React personnalisés ? Les lister avec leurs chemins"
```

**Vérifier la gestion des erreurs :**
```bash
gemini -p "@src/ @app/ La gestion d'erreurs est-elle implémentée pour toutes les pages ?"
```

**Vérifier l'implémentation Framer Motion :**
```bash
gemini -p "@src/components/ @src/app/ Quels composants utilisent Framer Motion ? Lister les animations implémentées"
```

**Vérifier la compatibilité mobile :**
```bash
gemini -p "@src/ @tailwind.config.ts Le design est-il responsive ? Analyser les breakpoints et les classes mobile"
```

### 8.3. Quand utiliser Gemini CLI

Utilise `gemini -p` quand :
- Analyse de codebases entières ou grands répertoires
- Comparaison de multiples gros fichiers
- Besoin de comprendre les patterns d'architecture du projet
- La fenêtre de contexte actuelle est insuffisante
- Travail avec des fichiers totalisant plus de 100KB
- Vérification d'implémentations spécifiques à l'échelle du projet
- Analyse des patterns de design et d'animation dans tout le projet

### 8.4. Notes importantes

- Les chemins dans la syntaxe @ sont relatifs au répertoire courant lors de l'invocation de gemini
- Le CLI inclura directement le contenu des fichiers dans le contexte
- Pas besoin du flag --yolo pour l'analyse en lecture seule
- La fenêtre de contexte de Gemini peut gérer des codebases entières qui déborderaient du contexte d'autres assistants
- Lors de la vérification d'implémentations, soyez spécifique sur ce que vous cherchez pour obtenir des résultats précis

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rachk02)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rachk02)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
