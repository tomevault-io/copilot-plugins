## base-react-template

> L'objectif est de générer une base de site web moderne, modulaire, et hautement performante en react/vite. Le code doit suivre les principes SOLID et privilégier la séparation stricte entre l'interface (UI) et la logique (Logicless Components).

# Blueprint: Architecture React Ultra-Optimisée (Full-Stack Ready) - 2026

## 1. Vision Générale

L'objectif est de générer une base de site web moderne, modulaire, et hautement performante en react/vite. Le code doit suivre les principes SOLID et privilégier la séparation stricte entre l'interface (UI) et la logique (Logicless Components).

## 2. Stack Technique (Core)

Secteur,Technologie,Points clés
Framework,React Router v7,"Mode Framework (ex-Remix), SSR/SSG natif."
Runtime,React 19,"Support des Actions, use API et Suspense."
Build Tool,Vite.js,"Plugin RR7, HMR ultra-rapide, build optimisé."
Styling,Tailwind CSS v4,"Container Queries, cascade simplifiée, CSS natif."
State,Zustand,État global léger pour l'UI et la session client.
Data Fetching,TanStack Query,Cache serveur/client et sync temps réel.
I18N,i18next,Détection serveur (SEO) + switch client sans reload.
Animations,Framer Motion,Transitions de routes et micro-interactions.

## 3. Architecture de Projet (Feature-Based)

/
├── app/
│ ├── root.tsx # Entrée principale (Providers, I18N, SEO global)
│ ├── routes.ts # Définition centralisée des routes
│ ├── entry.server.tsx # Config du rendu côté serveur
│ ├── entry.client.tsx # Config de l'hydratation côté client
│ ├── routes/ # Pages et Layouts
│ │ ├── \_public.tsx # Layout SEO (Navbar, Footer)
│ │ ├── \_public.articles.tsx # Blog/Articles (Prerendered)
│ │ ├── \_auth.tsx # Layout Login/Register
│ │ └── dashboard.\_index.tsx # App privée (Client-only)
│ ├── components/
│ │ ├── ui/ # Atomes (Boutons, Inputs via Tailwind v4)
│ │ └── features/ # Molécules métier (Comparateur, Graphiques)
│ ├── lib/
│ │ ├── i18n/ # Config i18next (server & client)
│ │ └── query-client.ts # Singleton TanStack Query
│ └── store/ # Stores Zustand (authStore, uiStore)
├── public/
│ ├── locales/ # Fichiers JSON de traduction (fr/en)
│ └── assets/ # Images et polices statiques
├── vite.config.ts # Configuration Vite + RR7 + Tailwind
└── tsconfig.json # Config TypeScript stricte (Path Aliases)

1. Vision Générale
   L'objectif est de générer une base de site web moderne, modulaire, et hautement performante. Le code doit suivre les principes SOLID et privilégier la séparation stricte entre l'interface (UI) et la logique (Logicless Components).
   . Remplacer useEffect par les Loaders
   Configuration Internationalisation (I18N)
   L'i18n est gérée en deux temps pour garantir un SEO parfait :

Serveur : Détection de la langue via URL, Cookie ou Header Accept-Language.

Client : Hydratation du dictionnaire et changement de langue instantané.

Utilisation dans un composant :

TypeScript

import { useTranslation } from "react-i18next";

export function PricingCard() {
const { t } = useTranslation();
return <button>{t("common.compare_prices")}</button>;
}
⚡ Flux de Données & Rendu

1. Rendu Hybride (SSR & Prerendering)
   Vitrine & Blog (SEO Priority) : Utilisation des loaders natifs de RR7. Les données sont récupérées côté serveur.

Zod Validation : Chaque retour d'API est validé via un schéma Zod avant d'atteindre le composant pour garantir l'intégrité du typage.

Résultat : Premier octet (TTFB) ultra-rapide et Indexation Google 100% fiable.

Dashboard (App Mode) : Utilisation du mode clientLoader pour les parties de l'application ne nécessitant pas de SEO, permettant une navigation instantanée type SPA sans solliciter le serveur inutilement.

2. Mutations & Formulaires (Progressive Enhancement)
   React 19 Actions : Remplacement des gestionnaires onSubmit classiques par les Actions de RR7.

Conform & Zod : Utilisation de la bibliothèque Conform pour la gestion des formulaires.

Validation côté serveur avec retour d'erreurs d'accessibilité immédiat.

Gestion native des états pending (chargement) sans besoin de useState manuel.

Note : Fonctionne même si le JavaScript n'est pas encore chargé.

3. Synchronisation & Cache (TanStack Query)
   Usage Sélectif : TanStack Query n'est plus utilisé pour le fetch initial (géré par les loaders), mais uniquement pour :

Polling/Temps Réel : Rafraîchissement des données du dashboard (ex: prix, notifications) en arrière-plan.

Optimistic UI : Mise à jour instantanée de l'interface avant confirmation serveur sur les actions complexes.

Persistance : Cache global partagé entre les routes pour éviter les fetchs redondants sur les ressources statiques.

4. Internationalisation (I18N)
   Détection : La langue est résolue dans le loader de la route root.tsx (via URL ou Header).

Zero-Flash : Le dictionnaire de traduction est injecté dans le contexte SSR, évitant tout "Layout Shift" ou texte non traduit au premier affichage.

♿ Accessibilité (A11y) & Qualité

- Respect strict des normes WCAG.
- Utilisation systématique des attributs ARIA.
- Gestion parfaite du focus clavier sur les éléments interactifs.
- Utilisation de Error Boundaries pour isoler les défaillances de composants.

🔍 SEO & Performance

- Intégration de JSON-LD pour chaque page (Articles, Organisation, Breadcrumbs).
- Utilise l'export meta de RR7.
- Mobile-First : Design responsive pensé d'abord pour le mobile.
- Utilisation de l'unité svh pour les Full-page Headers (évite les sauts sur mobile).
- Optimisation des images et Lazy-loading des composants lourds.

🎨 Design & Thémisation

- Dark/Light Mode : Gestion via variables CSS natives injectées par le ThemeProvider.
- Détection automatique de la préférence système de l'utilisateur.
- Utilisation des Container Queries (@container) pour des composants réellement autonomes.

5. Spécifications des Pages
   Chaque page doit être générée avec ses méta-données SEO et ses transitions Framer Motion :

- Home : Full-page Header immersif, sections modulaires.
- Articles : Liste dynamique, filtrable, avec Skeleton Loaders.
- Contact : Formulaire typé avec Zod, gestion des états (idle/loading/success/error).
- Login : Interface épurée, validation temps réel.
- Dashboard : Espace loggué avec Sidebar, Header persistant et contenu dynamique.
- A propos : Structure narrative et présentation d'équipe.
- 404 : Page stylisée avec animation et bouton de retour intelligent.

6. Livrables attendus
   Code source complet avec typage TypeScript Strict.

- Fichiers de configuration (tsconfig.json, vite.config.ts).
- Fichier README.md détaillant l'installation (npm install) et l'architecture.
- Structure de dossiers prête à l'emploi.

Une petite astuce "Pro-tip" pour ton implémentation :
Dans ton root.tsx, pense à utiliser le composant <ScrollRestoration /> de RR7. C'est un détail souvent oublié mais crucial pour l'expérience utilisateur, surtout sur les pages "Articles" où l'utilisateur revient en arrière depuis un contenu long.

C'est une excellente question. En 2026, pour qu'un agent IA (comme moi ou un agent de codage autonome) génère du code qui respecte exactement ton blueprint sans dériver vers des vieilles méthodes (comme useEffect ou axios), tu as besoin d'un fichier de contexte spécifique.

Je te conseille d'ajouter une section "7. Instructions pour l'Agent IA (System Prompting)" à ton MD ou de créer un fichier .cursorrules (si tu utilises Cursor) ou ai-instructions.md.

Voici la directive de configuration à ajouter pour "brider" l'IA sur ton architecture :

7. Instructions pour l'Agent IA (Guidelines)
   "En tant qu'expert React Router v7, tu dois suivre ces règles strictes pour toute génération de code :

Data Fetching : Interdiction d'utiliser useEffect pour le fetch. Utilise exclusivement les loaders (SSR) ou clientLoader (SPA).

Types : Utilise TypeScript en mode strict. Chaque retour de loader doit être typé via useLoaderData<typeof loader>.

Validation : Tout paramètre d'URL ou corps de formulaire doit être validé par un schéma Zod.

Formulaires : Utilise les Actions de RR7 avec la bibliothèque Conform. Pas de gestion d'état useState manuelle pour les inputs.

Styling : Utilise exclusivement les utilitaires Tailwind CSS v4. Priorise les Container Queries (@container) pour les composants de la galerie components/features.

Composants : Applique le pattern Logicless UI. Les composants dans ui/ ne doivent contenir aucune logique métier.

SEO : Chaque route doit exporter une fonction meta utilisant les données du loader pour le titre et les balises OpenGraph."

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fredconv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
