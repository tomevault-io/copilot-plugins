## karalix-page

> **Projet** : karalix-page

# Claude.md - karalix-page

## Vue d'ensemble du projet

**Projet** : karalix-page
**Type** : Site portfolio personnel
**Propriétaire** : Alix Ducros (krlx)
**URL** : https://krlx.fr
**Langue principale** : Français
**Repository** : github.com/Karalix/karalix-page

Site personnel servant de portfolio professionnel, vitrine académique, blog personnel et archive de matériel pédagogique pour Alix Ducros, Lead Frontend Developer et ancien chercheur en IHM (Interaction Homme-Machine).

## Stack technique

### Core
- **VitePress 1.3.4** - Générateur de site statique basé sur Vue 3
- **Vue 3.5.10** - Framework JavaScript (Composition API)
- **TypeScript** - Utilisé dans les fichiers de thème (.mts, .ts)
- **Vite** - Outil de build (via VitePress)

### Styling
- **Tailwind CSS 3.4.13** - Framework CSS utility-first
- **DaisyUI 4.12.11** - Bibliothèque de composants pour Tailwind
- **@tailwindcss/typography 0.5.15** - Plugin pour le contenu prose
- **PostCSS 8.4.47** - Traitement CSS
- **Autoprefixer 10.4.20** - Préfixes vendeurs CSS

### Animation
- **GSAP 3.12.5** - Bibliothèque d'animation pour le curseur personnalisé

## Structure du projet

```
/
├── .vitepress/           # Configuration VitePress et thème
│   ├── config.mts        # Configuration principale
│   ├── theme/            # Thème personnalisé
│   │   ├── Layout.vue    # Composant de layout principal
│   │   ├── index.ts      # Point d'entrée du thème
│   │   ├── style.css     # Styles globaux (imports Tailwind)
│   │   └── notes.data.mts # Content loader pour les notes
│   └── cache/            # Cache de build
├── cours/                # Matériel pédagogique (UML, dev web)
├── notes/                # Articles de blog (16 fichiers markdown)
├── public/               # Assets statiques (PDFs, images, ~47MB)
├── icons/                # Icônes réseaux sociaux et projets
├── index.md              # Page d'accueil
├── cv.md                 # Curriculum Vitae
├── projects.md           # Projets portfolio
├── publications.md       # Publications académiques
├── library.md            # Liste de lecture partagée
├── package.json          # Dépendances
├── tailwind.config.js    # Configuration Tailwind
└── postcss.config.js     # Configuration PostCSS
```

### Répertoires clés

- **`.vitepress/theme/`** : Thème Vue personnalisé avec layout grille, curseur personnalisé, styles
- **`notes/`** : Articles de blog (tutoriels techniques, recettes, gaming, observations personnelles)
- **`cours/`** : Matériel de cours universitaire (UML, TP web dev 2017-2020)
- **`public/`** : Assets statiques (~47MB) - PDFs, screenshots, CV, photos

## Conventions de code

### Nommage des fichiers
- **Markdown** : kebab-case (`cci-uml-2018-tp1.md`)
- **Composants Vue** : PascalCase (`Layout.vue`)
- **Fichiers config** : kebab-case avec extensions (`.config.js`, `.data.mts`)

### Frontmatter

```yaml
---
title: "Titre de la page"
layout: page|article|home  # Type de layout
date: "YYYY-MM-DD"          # Pour les articles
notelist: true              # Pour l'index des notes
---
```

**Layouts disponibles** :
- `home: true` - Page d'accueil avec grille personnalisée
- `layout: page` - Pages standard en prose
- `layout: article` - Articles de blog avec bio d'auteur
- `notelist: true` - Liste auto-générée des notes

### Patterns Vue/TypeScript

- **Composition API** avec syntaxe `<script setup>`
- **TypeScript** pour le type-safety (voir `notes.data.mts`)
- **Refs réactifs** : `ref()`
- **Lifecycle hooks** : `onMounted`, `onUpdated`
- **Composables VitePress** : `useData()`, `createContentLoader()`

### Patterns CSS

- **Tailwind utility classes** utilisées extensivement
- **Composants DaisyUI** : `card`, `card-body`, `btn`, `prose`
- **Effets hover** : `hover:shadow-xl`, `hover:translate-x-4`, utilities `group`
- **Design responsive** : breakpoints `sm:`, `md:`
- **Layouts grid** : `grid grid-cols-1 sm:grid-cols-3 md:grid-cols-4`

### Patterns d'animation

- **GSAP** : `gsap.to()` pour animations fluides
- **Animations event-driven** sur mousemove et hover
- **Cursor tracking** avec positionnement absolu et transforms

## Commandes de développement

```bash
# Serveur de développement local
npm run docs:dev

# Build production
npm run docs:build

# Preview du build production
npm run docs:preview
```

**Workflow** :
1. Développement : `npm run docs:dev` (HMR activé)
2. Build : `npm run docs:build` (sortie dans `.vitepress/dist/`)
3. Preview : `npm run docs:preview`

## Fonctionnalités principales

### 1. Page d'accueil avec grille Bento
- Layout grille/masonry avec Tailwind CSS
- Card de profil avec photo
- Liens rapides (CV, Library, Publications, Projects, Notes)
- Liens externes (Hormur, Kronikle)
- Réseaux sociaux (GitHub, LinkedIn, HAL, Mastodon)
- Contact (email, Signal)

### 2. Curseur personnalisé animé
- GSAP pour suivi fluide de la souris
- Effet `mix-blend-difference` pour visibilité
- Scale au hover des liens
- Curseur par défaut désactivé (`cursor: none`)

### 3. Système de notes dynamique
- Content loader TypeScript (`notes.data.mts`)
- Extraction frontmatter (titre, date)
- Tri par date (plus récent en premier)
- Formatage dates en locale française
- Affichage des 4 notes les plus récentes sur homepage

### 4. Sections de contenu
- **CV** : Timeline professionnel (2012-présent)
- **Projects** : Portfolio de projets (Bibliotouch, Explore, Hormur, My Tiny Café, etc.)
- **Publications** : Papers académiques (2018-2024) avec DOIs et PDFs
- **Notes** : Blog (recettes, tutoriels tech, gaming, tips)
- **Library** : Liste de lecture avec notations et liens ebooks
- **Cours** : Matériel pédagogique universitaire

## Contexte professionnel

**Identité** : Lead Frontend Engineer, ancien doctorant en IHM
**Rôle actuel** : Hormur (startup)
**Freelance** : AlphaValue (2025)
**Background** : PhD en IHM (2017-2021) sur interfaces tactiles en bibliothèque
**Focus technique** : Nuxt, Vue, PWA, UX design, TypeScript

**Projets notables** :
1. My Tiny Café (2025) - PWA personnelle pour commandes familiales
2. Hormur (2022-présent) - Plateforme arts et culture
3. Habilitrak (2023) - Tracker d'habilitations employés
4. Explore 2 (2019-2021) - Projet de recherche bibliothèques
5. Bibliotouch (2017) - Interface bibliothèque interactive

**Contributions académiques** :
- 5+ publications peer-reviewed (CHI, DIS, ISS)
- Focus : affichages publics, bibliothèques, IHM
- Collaborations avec institutions de recherche européennes

## Langue et localisation

- **Langue principale** : Français
- **Langue secondaire** : Anglais (notes techniques, publications)
- **Formatage dates** : Locale française (`toLocaleDateString('fr-FR')`)
- **Contenu** : Bilingue selon contexte

## Bonnes pratiques pour contribuer

### Lors de modifications
1. **Lire avant modifier** : Toujours lire les fichiers existants avant de proposer des changements
2. **Respecter les conventions** : Suivre les patterns de nommage et de code existants
3. **Éviter l'over-engineering** : Faire uniquement les changements demandés
4. **Sécurité** : Attention aux vulnérabilités (XSS, injection, OWASP top 10)
5. **Langue** : Respecter le français pour le contenu principal

### Ajout de contenu
- **Notes** : Créer dans `/notes/` avec frontmatter (title, date, layout: article)
- **Pages** : Créer à la racine avec frontmatter approprié
- **Assets** : Placer dans `/public/` pour références statiques
- **Cours** : Ajouter dans `/cours/` si matériel pédagogique

### Style et formatting
- Utiliser **Tailwind utilities** plutôt que CSS custom
- Privilégier **composants DaisyUI** quand approprié
- Conserver **design responsive** avec breakpoints
- Tester avec `npm run docs:dev` avant commit

## Implémentations techniques notables

### Système de curseur personnalisé
- Design dual-cursor (grand et petit)
- Suivi fluide avec GSAP
- Mix-blend-difference pour contraste
- Pointer-events-none pour éviter interférences
- Animation scale au hover des liens

### Chargement de données notes
- Interface TypeScript pour type safety
- Extraction et parsing frontmatter
- Formatage automatique des dates (locale FR)
- Tri par date descendant
- Filtrage des entrées valides

### Layout grille responsive
- CSS Grid avec auto-rows
- Contrôles span pour contenu featured
- Effets hover avec utilities group
- Breakpoints mobile-first

### Intégration OpenSearch
- Descripteur XML pour Mistral AI
- Compatible Firefox
- Démontre connaissance standards web avancés

## Notes importantes

- **Taille projet** : ~106MB (dont ~47MB assets publics)
- **Dépendances** : Stack minimaliste et focalisée
- **Build output** : `.vitepress/dist/` (ignoré par git)
- **Node modules** : Ignorés par git
- **Branches** : Suivre workflow git avec branches `claude/` pour développement

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Karalix)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Karalix)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
