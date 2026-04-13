## capriati-sa-nextjs

> ![MIT License](https://img.shields.io/badge/license-MIT-blue)

# 📚 Capriati S.A. – Peinture & Décoration

![MIT License](https://img.shields.io/badge/license-MIT-blue)
[![TypeScript](https://badgen.net/badge/icon/typescript?icon=typescript&label)](https://typescriptlang.org)
![ESLint](https://img.shields.io/badge/code%20style-eslint-brightgreen)

Site web de l’entreprise de peinture Capriati S.A., développé avec Next.js 15 et TypeScript.

## 🚀 Technologies

- **Framework**: [Next.js 15](https://nextjs.org/)
- **Langage**: [TypeScript](https://www.typescriptlang.org/)
- **Styling**: [Tailwind CSS V4](https://tailwindcss.com/)
- **UI Components**: [Shadcn UI](https://ui.shadcn.com/)
- **Animations**: Framer Motion
- **CMS**: [Sanity](https://www.sanity.io/)
- **Email**: [Resend](https://resend.com/)
- **Déploiement**: [Vercel](https://vercel.com/)

## 🛠️ Technologies Incluses

- **Next.js 15**
- **React 19**
- **TypeScript 5**
- **ESLint 9**
- **Prettier 3**
- **Tailwind CSS 4**
- **Shadcn UI**
- **App Directory**
- **System, Light & Dark Mode**
- **Next.js Bundle Analyzer**

### 🔧 Plugins ESLint

- [**@eslint/js**](https://www.npmjs.com/package/@eslint/js) - Configuration ESLint moderne
- [**typescript-eslint**](https://github.com/typescript-eslint/typescript-eslint) - Support TypeScript
- [**eslint-plugin-react**](https://github.com/jsx-eslint/eslint-plugin-react) - Règles React
- [**@next/eslint-plugin-next**](https://github.com/vercel/next.js) - Règles Next.js
- [**eslint-config-prettier**](https://github.com/prettier/eslint-config-prettier) - Compatibilité Prettier
- [**eslint-plugin-tailwindcss**](https://github.com/francoismassart/eslint-plugin-tailwindcss) - Règles Tailwind
- [**eslint-plugin-import**](https://github.com/import-js/eslint-plugin-import) - Gestion des imports
- [**eslint-plugin-promise**](https://github.com/eslint-community/eslint-plugin-promise) - Gestion des promesses

### ✨ Plugins Prettier

- [**@trivago/prettier-plugin-sort-imports**](https://github.com/trivago/prettier-plugin-sort-imports) - Tri automatique des imports
- [**prettier-plugin-tailwindcss**](https://github.com/tailwindlabs/prettier-plugin-tailwindcss) - Formatage Tailwind

## 🎯 Principes Fondamentaux

1. **Performance**

   - SSG (Static Site Generation) pour toutes les pages
   - Optimisation des images avec`next/image`
   - Revalidation ISR pour le contenu dynamique

2. **Accessibilité**

   - Respect des normes WCAG 2.1
   - Navigation au clavier optimisée
   - Zoom de la typographie fluid avec fallbacks

3. **UX/UI**

   - Design responsive
   - Animations fluides avec Framer Motion

## 📁 Structure du Projet

```md
📦 capriati-sa/
├── 📚 .cursor/                # Règles, conventions, doc interne (Markdown)
│   └── 📂 rules/
├── 📂 public/                 # Fichiers statiques (images, favicon, etc.)
├── 📂 src/
│   ├── 📂 app/                # Routing Next.js 15 (pages, layouts, templates)
│   │   ├── 📝 page.tsx        # Accueil
│   │   ├── 📝 layout.tsx      # Layout principal
│   │   ├── 📂 about/          # Page À propos
│   │   ├── 📂 contact/        # Page Contact
│   │   ├── 📂 services/       # Catalogue des services
│   │   │   ├── 📂 [slug]/     # Pages dynamiques pour chaque service
│   │   ├── 📂 works/          # Réalisations (galerie, projets)
│   │   │   ├── 📂 [slug]/     # Pages dynamiques pour chaque réalisation
│   │   └── 📝 globals.css     # Styles globaux
│   ├── 🧩 components/
│   │   ├── 🧩 ui/             # Composants Shadcn UI (ne pas modifier)
│   │   ├── 🧩 shared/         # Composants réutilisables (Header, Footer, etc.)
│   │   └── 🧩 pages/          # Composants spécifiques à une page
│   ├── 🔧 lib/                # Fonctions utilitaires, config Sanity, hooks, etc.
│   │   ├── 🔧 sanity/         # Config et clients Sanity
│   │   ├── 🔧 resend/         # Configuration de resend pour les formulaires
│   │   └── 🔧 utils/          # Fonctions utilitaires diverses
│   ├── 🎨 styles/             # Fichiers CSS/variables complémentaires
│   ├── 🔷 types/              # Types TypeScript partagés
│   ├── 🎣 hooks/              # Hooks de l'application
│   └── ✅ tests/              # Tests unitaires et d’intégration
├── 📝 .env.local              # Variables d’environnement (non versionné)
├── 📝 .eslintrc.json          # Config ESLint
├── 📝 .prettierrc             # Config Prettier
├── tailwind.config.ts      # Config Tailwind
├── 📝 next.config.mjs         # Config Next.js
├── 📝 package.json
├── 📝 pnpm-lock.yaml
└── 📝 README.md
```

## 📝 Conventions de Code

1. **Nommage**

   - Composants : PascalCase
   - Fonctions : camelCase
   - Types : PascalCase
   - Variables : camelCase

2. **Organisation**

   - Un composant par fichier
   - Types dans des fichiers séparés
   - Styles avec Tailwind CSS

3. **Documentation**

   - JSDoc pour les fonctions complexes
   - README dans chaque dossier important
   - Commentaires en français

## 🔄 Workflow Git

1. **Branches**

   - `main` : Production

     - 🔒 Protection contre les push directs
     - 🔍 Pull Request requis
     - 👥 Reviews approuvées requises (1 minimum)
     - ✅ Status checks requis
     - 🏷️ Tags sémantiques obligatoires

   - `develop` : Développement

     - 🔍 Pull Request requis
     - 👥 Reviews approuvées requises (1 minimum)
     - ✅ Status checks requis

   - `feature/*` : Nouvelles fonctionnalités
   - `fix/*` : Corrections de bugs
   - `docs/*` : Documentation

2. **Commits**

   - Format :`type(scope): description`
   - Types : feat, fix, docs, style, refactor, test, chore
   - Messages en français

3. **Protection des Branches**

   - Configuration dans `.github/workflows/branch-protection.yml`
   - Documentation dans `.github/README.md`
   - Workflows automatisés pour la validation
   - Règles de sécurité et de qualité

## 🎨 Design System

1. **Couleurs**

   - Accent : #EE332D (Rouge)
   - Gray : #F9A542 (orange)

2. **Typographie**

   - Système fluide basé sur clamp()
   - Échelle : xs, sm, base, lg, xl, 2xl, 3xl
   - Espacement fluide pour le line-height

3. **Espacement**

   - Base fluide avec clamp()
   - Variables CSS pour les espacements
   - Classes utilitaires : p-fl-_, m-fl-_, gap-fl-\*

## 📱 Responsive Design

1. **Breakpoints**

   - Mobile : < 640px
   - Tablet : 640px - 1024px
   - Desktop : > 1024px

2. **Approche**

   - Mobile First
   - Progressive Enhancement
   - Typographie et espacements fluides

## 🔗 Liens vers la Documentation

- 📚 [`README.md`](./.cursor/rules/README.md) - Guide principal des règles
- 📝 [`.cursorrules.md`](./.cursorrules.md) - Vue d'ensemble des règles du projet
- 🔧 [`code-style.md`](./.cursor/rules/code-style.md) - Conventions de style de code
- 🧩 [`components.md`](./.cursor/rules/components.md) - Règles pour les composants React
- 💾 [`git-workflow.md`](./.cursor/rules/git-workflow.md) - Convention de Git et Gitub
- 🎨 [`project-design.md`](./.cursor/rules/project-design.md) - Système de design et UI
- 🔷 [`typescript.md`](./.cursor/rules/typescript.md) - Conventions TypeScript
- 📝 [`markdown-style.md`](./.cursor/rules/markdown-style.md) - Conventions de style Markdown

## 🔄 Gestion des Fichiers Cursor

### 1. 📝 Ajout d'un Nouveau Fichier Cursor

Lors de l'ajout d'un nouveau fichier de règles Cursor :

1. **Création du fichier**

   - Utiliser le format Markdown
   - Suivre la structure standard
   - Inclure les emojis appropriés
   - Documenter clairement les règles

2. **Mise à jour des références**

   - Ajouter le lien dans `.cursorrules.md`
   - Mettre à jour le README correspondant
   - Vérifier les dépendances

3. **Validation**

   - Vérifier la cohérence avec les autres règles
   - S'assurer de la clarté des exemples
   - Valider les liens et références

# Règles pour l'Assistant IA

## Dossiers protégés

- `src/components/ui/` Ne pas toucher les fichier Shadcn originaux
- `.ressources/` (contient les fichiers originaux et les templates)

### Raisons

- Ces dossiers contiennent des composants et configurations de base
- Toute modification peut casser la cohérence du système
- Les mises à jour de shadcn/ui seraient compromises
- Le dossier .ressources contient le repository original et des ressources importantes

## Utilisation Correcte

### 🔍 Analyse du projet

Lors de la demande de l'utilisateur pour une analyse du projet inspection obligatoire de tous les dossiers et fichiers.

Ci-dessous, la liste des dossiers avec leur contenu est à ignorer
DOSSIERS:

- .next/
- .ressources/
- .sanity/
- node_modules/

✅ À FAIRE :

- Utiliser les composants existants tels quels
- Importer depuis ces dossiers
- Créer de nouveaux composants qui utilisent ces composants de base

❌ À NE PAS FAIRE :

- Modifier les fichiers dans ces dossiers
- Dupliquer ou renommer ces composants
- Suggérer des modifications directes à ces fichiers

## En cas de besoin de modification

Si des modifications sont nécessaires dans ces dossiers :

1. En informer l'utilisateur
2. Laisser l'utilisateur gérer les modifications
3. Attendre la confirmation avant de continuer

### Commits de Refactoring avec Backup

Pour les refactorisations majeures, créer un commit de backup avec :

- Type : `refactor(backup)`
- Message principal : Description concise de la sauvegarde
- Corps du message : Liste des changements prévus
- Pour les détails et formatage, voir [`git-workflow.md`](./.cursor/rules/git-workflow.md/# 4. Backups de code)

## 🧪 Tests (Avec Vitest ou Jest qui viendront plus tard)

1. **Configuration**

   - Utilisation de Vitest pour les tests
   - Support natif de TypeScript
   - Tests unitaires et d'intégration
   - Interface utilisateur de test intégrée

2. **Organisation**

   - Tests placés dans le dossier `tests/`
   - Structure miroir du code source
   - Fichiers de test avec suffixe `.test.ts` ou `.test.tsx`

3. **Conventions**

   - Tests descriptifs avec `describe` et `it`
   - Utilisation de `@testing-library/react` pour les composants
   - Mocks avec les fonctionnalités natives de Vitest
   - Coverage avec V8

4. **Scripts**

   - Contient les scripts de base Next.js
   - Contient des scripts pour Sanity
   - Contient des scripts de formatage
   - Contient des scripts liés à de la maintenance et developpement complémentaires

> Pour de plus amples informations sur les scripts, consultez le [README.md](./README.md) principal.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Pataco80) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
