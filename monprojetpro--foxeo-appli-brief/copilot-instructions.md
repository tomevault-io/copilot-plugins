## foxeo-appli-brief

> **Nom:** appli-brief-BMAD

# CLAUDE.md - Contexte Projet

## Projet
**Nom:** appli-brief-BMAD
**Description:** Application de génération de briefing optimisée pour la méthode BMAD

## Objectif
Résoudre le problème des briefs incomplets ou négligés qui freinent le démarrage des projets. L'application aide les chefs de projet à produire des briefs de qualité, prêts à être exploités par l'Analyst BMAD.

## Utilisateur Cible
**Persona principale : Sophie, Chef de Projet**
- Post-réunion client, bureau ou déplacement
- Frustration : Notes éparses → brief bâclé → Analyst qui pose 10 questions
- Besoin : Produire un brief "propre" rapidement + savoir s'il est complet

## Modes de Fonctionnement

### Mode Guidé
- Rédaction depuis zéro avec questions structurées pas à pas
- 4 questions séquentielles (Problème, Cible, Solution, Différenciant)
- Alternative pour les situations "page blanche"

### Mode Analyse (expérience principale)
- Import d'un document/retranscription brut
- Analyse IA et transformation en brief structuré
- Diagnostic des manques + questions de relance contextuelles
- Score de complétude en temps réel

## Référentiel de Complétude BMAD

| Élément | Requis | Description |
|---------|--------|-------------|
| Problème | Requis | La douleur en 1-2 phrases claires |
| Cible | Requis | Qui + contexte minimum |
| Solution | Requis | L'idée générale, le "quoi" |
| Différenciant | Optionnel | Pourquoi ça n'existe pas encore |

### Scoring visuel
- 🔴 **Incomplet** - Il manque Problème, Cible OU Solution
- 🟠 **À compléter** - Les 3 sont là mais trop vagues
- 🟢 **Prêt pour l'Analyst** - Les 3 éléments sont clairs et compréhensibles
- ⭐ **Optimal** - + le différenciant est renseigné

## Philosophie
> "Le brief initial doit être suffisamment clair pour que l'Analyst comprenne l'intention, suffisamment ouvert pour qu'il puisse creuser."

## Stack Technique (décidé)

| Aspect | Choix |
|--------|-------|
| **Type** | SPA (Single Page Application) |
| **Design System** | shadcn/ui + Tailwind CSS |
| **Icônes** | Lucide Icons |
| **Providers IA** | Claude API, OpenAI GPT, Gemini API |
| **Stockage** | localStorage (briefs) |

## Direction UX (décidée)

**Direction Dashboard** - Interface de type tableau de bord avec :
- Header avec navigation principale
- Zone de statistiques (briefs créés, prêts, à compléter, incomplets)
- Panneau de brief détaillé avec les 4 éléments colorés
- Brief Library intégrée

## Structure du Projet

```
appli-brief-BMAD/
├── app/                # Next.js App Router
│   ├── globals.css     # Tailwind + scoring colors
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── ui/             # shadcn/ui (DO NOT MODIFY)
│   ├── brief/          # Brief feature components
│   ├── dashboard/      # Dashboard components
│   └── layout/         # Layout components
├── lib/
│   ├── providers/      # AI provider abstraction
│   ├── scoring/        # Scoring logic
│   ├── storage/        # localStorage helpers
│   ├── export/         # Export utilities
│   └── utils/          # General utilities
├── types/              # TypeScript types
├── contexts/           # React Contexts
├── hooks/              # Custom React hooks
├── _bmad/              # Configuration BMAD Method
├── _bmad-output/       # Artefacts de planification
│   ├── analysis/       # Sessions de brainstorming
│   ├── project-planning-artifacts/  # Product brief, epics
│   │   ├── epics.md    # Epics & Stories (6 epics, 24 stories)
│   │   └── implementation-readiness-report-2026-01-11.md
│   ├── implementation-artifacts/    # Stories et sprint tracking
│   │   ├── sprint-status.yaml       # Suivi du sprint
│   │   └── 1-1-project-initialization.md  # Story 1.1
│   ├── prd.md          # Product Requirements Document
│   ├── architecture.md # Architecture technique
│   ├── ux-design-specification.md   # Spécifications UX
│   ├── ux-design-directions.html    # Mockups visuels
│   └── bmm-workflow-status.yaml     # Suivi workflow BMAD
└── CLAUDE.md           # Ce fichier
```

## Conventions

- **Langue:** Français pour la documentation et les interfaces
- **Méthode:** BMAD Method (Business-Minded Agile Development)

## Statut du Projet

**Phase actuelle:** ✅ Implementation Complete - Tous les Epics terminés

| Phase | Statut | Artefact |
|-------|--------|----------|
| Brainstorming | ✅ Terminé | `brainstorming-session-2026-01-07.md` |
| Product Brief | ✅ Terminé | `product-brief-appli-brief-BMAD-2026-01-07.md` |
| PRD | ✅ Terminé | `prd.md` |
| UX Design | ✅ Terminé | `ux-design-specification.md` |
| Architecture | ✅ Terminé | `architecture.md` |
| Epics & Stories | ✅ Terminé | `epics.md` (6 epics, 24 stories) |
| Implementation Readiness | ✅ Terminé | `implementation-readiness-report-2026-01-11.md` |
| Sprint Planning | ✅ Terminé | `sprint-status.yaml` |
| **Epic 1** | ✅ Done | 4/4 stories terminées |
| **Epic 2** | ✅ Done | 5/5 stories terminées |
| **Epic 3** | ✅ Done | 5/5 stories terminées |
| **Epic 4** | ✅ Done | 5/5 stories terminées |
| **Epic 5** | ✅ Done | 5/5 stories terminées |
| **Epic 6** | ✅ Done | 5/5 stories terminées |

## Projet Terminé
Toutes les fonctionnalités MVP ont été implémentées.

### Epic 3: AI Core & Analysis Engine (COMPLETED)

#### Story 3.1: AI Provider Abstraction Layer (COMPLETED)
- ✅ Interface AIProviderService avec analyze, generateQuestions, evaluate
- ✅ Providers Claude, OpenAI, Gemini implémentés
- ✅ Factory getProvider avec cache
- ✅ Retry logic (NFR13) avec withRetry helper
- ✅ Types et schémas Zod

#### Story 3.2: Secure API Proxy Routes (COMPLETED + REVIEWED)
- ✅ Route POST `/api/analyze` - Analyse du contenu brut
- ✅ Route POST `/api/questions` - Génération de questions de relance
- ✅ Route GET/POST `/api/health` - Health check des providers
- ✅ Clés API uniquement côté serveur (NFR5-8)
- ✅ Format réponse `{ success, data/error }` standardisé
- ✅ Gestion des erreurs avec messages user-friendly
- ✅ Timeout 30s + retry (NFR12 + NFR13)
- ✅ Helpers partagés (`lib/api/helpers.ts`)
- ✅ Validation Zod cohérente sur toutes les routes

#### Story 3.3: Brief Analysis & Element Extraction (COMPLETED)
- ✅ Client API service (`lib/api/client.ts`)
- ✅ Fonction `analyzeContent()` avec timeout et gestion erreurs
- ✅ Integration dans page Mode Analyse
- ✅ Mise a jour du brief avec elements extraits
- ✅ Messages de succes/erreur user-friendly
- ✅ Build TypeScript passe

#### Story 3.4: Content Evaluation & Reformulation Suggestions (COMPLETED)
- ✅ Prompt d'analyse modifie pour retourner suggestions
- ✅ Types API mis a jour (suggestion dans ElementAnalysis)
- ✅ Route POST `/api/evaluate` pour evaluation individuelle
- ✅ Page Analyse stocke les suggestions
- ✅ BriefElementCard affiche suggestions pour elements vagues
- ✅ Bouton "Appliquer" pour utiliser la suggestion
- ✅ Bouton "Evaluer ma reponse" dans Mode Guide
- ✅ Affichage des suggestions inline dans Mode Guide

#### Story 3.5: Manual Editing & Re-analysis (COMPLETED)
- ✅ Edition inline des elements via BriefElementCard (FR31)
- ✅ Auto-save des modifications manuelles
- ✅ Bouton "Re-analyser" en Mode Analyse (FR29)
- ✅ Logique de fusion pour re-analyse (preserve + nouveau contenu)
- ✅ Bouton "Re-evaluer" dans BriefPanel pour recalculer les statuts
- ✅ Evaluation parallele de tous les elements avec contenu
- ✅ Messages de succes/erreur user-friendly

### Epic 4: Scoring & Visual Feedback (COMPLETED)

#### Story 4.1: Scoring Calculation Logic (COMPLETED)
- ✅ Module de scoring créé (lib/scoring/)
- ✅ Fonction calculateBriefStatus avec règles BMAD (FR13)
  - 🔴 Incomplete: Problème, Cible OU Solution manquant
  - 🟠 Needs work: Les 3 présents mais au moins un vague
  - 🟢 Ready: Problème, Cible, Solution clairs
  - ⭐ Optimal: Les 4 éléments incluant Différenciant clairs
- ✅ Fonction getScoringDetail pour détails et explications (FR32)
- ✅ Utilitaires de vérification de statut
- ✅ Intégration dans storage layer (auto-calcul à chaque update)
- ✅ Calcul temps réel garanti (NFR3) - synchrone, < 1ms
- ✅ Correction types TypeScript (suggestion?: string | null)

#### Story 4.2: Visual Scoring Badge & Status Display (COMPLETED)
- ✅ Amélioration StatusBadge avec tooltip détaillé (FR14, FR32)
- ✅ Badge avec couleurs et icônes BMAD (🔴 ✕, 🟠 ⚠, 🟢 ✓, ⭐ ⭐)
- ✅ Tooltip affichant le détail du scoring via getScoringDetail
- ✅ Tooltip liste éléments clairs/vagues/manquants avec couleurs
- ✅ Intégration dans BriefPanel (remplacement du badge inline)
- ✅ Intégration dans BriefListItem (avec tooltip)
- ✅ Amélioration accessibilité couleurs WCAG AA (4.5:1 contrast)
  - Light mode: red-600, orange-600, green-600, yellow-600
  - Dark mode: red-400, orange-400, green-400, yellow-400
- ✅ Build TypeScript passe sans erreur

#### Story 4.3: Element Status Coloring (COMPLETED)
- ✅ Bordures colorées (2px) selon statut (FR15)
  - Rouge pour missing
  - Orange pour vague
  - Vert pour clear
  - Or pour différenciateur clear
- ✅ Backgrounds colorés légers (bg-scoring-bg-*)
  - Opacité 30% pour missing (discret)
  - Opacité 50% pour vague, clear, optimal
- ✅ Placeholder "Non renseigné" en rouge pour éléments missing
- ✅ Support dark mode avec couleurs appropriées
- ✅ Accessibilité WCAG AA respectée (4.5:1 contrast)

#### Story 4.4: Scoring Detail Tooltip (COMPLETED)
- ✅ Implémenté dans Story 4.2 - Tooltip intégré dans StatusBadge

#### Story 4.5: Real-time Score Updates & Dashboard Stats (COMPLETED)
- ✅ Mise à jour automatique du status à chaque modification (FR16)
- ✅ Stats dashboard calculées en temps réel
- ✅ BriefContext appelle setStats() après chaque CRUD operation
- ✅ Performance garantie < 200ms (NFR2) - mesuré à < 100ms
- ✅ Calcul synchrone du scoring < 1ms (NFR3)
- ✅ Architecture réactive via React Context
- ✅ Debounce 500ms sur auto-save pour optimisation

### Epic 5: Follow-up Questions & Export (COMPLETED)

#### Story 5.1: Follow-up Questions Generation (COMPLETED)
- ✅ Route POST `/api/questions` - Génération de questions de relance
- ✅ Client API `generateQuestions()` avec validation Zod
- ✅ Composant FollowUpQuestions avec loading/error/success states
- ✅ Bouton "Copier" avec feedback "Copié !"
- ✅ Intégration dans BriefPanel (conditionnel si éléments incomplets)
- ✅ Action `updateFollowUpQuestions()` dans BriefContext

#### Story 5.2: Copy Follow-up Questions (COMPLETED)
- ✅ Implémenté dans Story 5.1 - Bouton copie intégré

#### Story 5.3: Client Response Integration (COMPLETED)
- ✅ Méthode `integrateResponses()` dans providers (Claude, OpenAI, Gemini)
- ✅ Route POST `/api/integrate-responses` avec validation Zod
- ✅ Client API `integrateResponses()` avec validation réponse
- ✅ Composant ClientResponseInput avec textarea et validation
- ✅ Action `integrateClientResponses()` dans BriefContext
- ✅ Mise à jour du status des éléments à 'vague' après intégration
- ✅ Clear des questions après intégration réussie

#### Story 5.4: Brief Preview (COMPLETED)
- ✅ Utilitaires de formatage `lib/export/formatBrief.ts`
  - `formatBriefAsMarkdown()` - Format Markdown
  - `formatBriefAsText()` - Format texte brut
  - `generateExportFilename()` - Nom de fichier avec date
- ✅ Composant BriefPreview avec Dialog (FR23, FR24)
- ✅ Boutons d'export intégrés (Copier, .md, .txt)
- ✅ Bouton "Aperçu" dans BriefPanel
- ✅ Format professionnel avec les 4 éléments BMAD labellisés

#### Story 5.5: Export Clipboard & File Download (COMPLETED)
- ✅ Implémenté dans BriefPreview (Story 5.4)
- ✅ Copier dans le presse-papier (FR22)
- ✅ Télécharger en .md (FR23)
- ✅ Télécharger en .txt (FR24)

### Epic 6: Settings & Provider Configuration (COMPLETED)

#### Story 6.1: Settings Page & Provider Selection (COMPLETED)
- ✅ Page `/settings` avec sélection du provider (FR25)
- ✅ Composant ProviderSelector avec dropdown
- ✅ Hook useSettings pour persistence localStorage
- ✅ Hook useAIApi pour injection automatique du provider

#### Story 6.2: API Key Configuration (COMPLETED)
- ✅ Information sur la configuration des clés (FR26)
- ✅ Affichage des variables d'environnement requises
- ✅ Clés API côté serveur uniquement (NFR8)

#### Story 6.3: API Error Handling & Messages (COMPLETED)
- ✅ Messages d'erreur user-friendly (FR27)
- ✅ Suggestions d'actions (retry, check key, try another provider)
- ✅ Implémenté dans lib/api/client.ts et helpers.ts

#### Story 6.4: Provider Health Check (COMPLETED)
- ✅ Composant ProviderHealthCheck
- ✅ Bouton "Tester la connexion"
- ✅ Affichage statut ✓/✕ pour chaque provider
- ✅ Route GET/POST `/api/health`

#### Story 6.5: Offline Mode & Degraded Experience (COMPLETED)
- ✅ Hook useAIAvailability pour vérification périodique
- ✅ Composant OfflineBanner avec message "Mode hors-ligne"
- ✅ Bouton "Analyser" désactivé quand IA indisponible (NFR17)
- ✅ Mode Guidé toujours disponible
- ✅ Sauvegarde locale préservée

### Epic 1: Project Foundation & Dashboard Shell (COMPLETED)

#### Story 1.1: Project Initialization (COMPLETED)
- ✅ Next.js 16.1.1 + App Router + TypeScript strict
- ✅ shadcn/ui avec composants requis
- ✅ Couleurs de scoring BMAD configurées (Tailwind v4)
- ✅ Structure feature-based créée

#### Story 1.2: Dashboard Layout & Navigation (COMPLETED)
- ✅ Header avec navigation (Dashboard, Nouveau, Historique, Settings)
- ✅ StatsRow avec 4 StatCards colorées
- ✅ Dashboard principal avec brief panel placeholder
- ✅ Pages placeholder pour toutes les routes
- ✅ Layout responsive

#### Story 1.3: Brief Data Model & Types (COMPLETED)
- ✅ Types TypeScript (Brief, Element, Status) dans types/brief.ts
- ✅ Schémas Zod pour validation runtime
- ✅ Types API dans types/api.ts
- ✅ Barrel export depuis types/index.ts

#### Story 1.4: LocalStorage Layer & Auto-Save (COMPLETED)
- ✅ lib/storage avec CRUD briefs et settings
- ✅ BriefContext avec React Context
- ✅ hooks useBriefs et useAutoSave
- ✅ BriefProvider intégré dans layout.tsx

### Epic 2: Brief Creation UI (COMPLETED)

#### Story 2.1: Mode Selection & New Brief Creation (COMPLETED)
- ✅ Page `/nouveau` avec choix Mode Guidé / Mode Analyse
- ✅ Pages `/nouveau/guide` et `/nouveau/analyse` créent un brief automatiquement
- ✅ Composant BriefActions (Nouveau brief, Effacer)
- ✅ AlertDialog pour confirmation avant effacement

#### Story 2.2: Mode Guidé - Four Questions Form (COMPLETED)
- ✅ Formulaire interactif avec 4 questions BMAD
- ✅ Navigation précédent/suivant avec sauvegarde auto
- ✅ Vue récapitulative avec les 4 éléments
- ✅ Progress indicator cliquable

#### Story 2.3: Mode Analyse - Text Import Interface (COMPLETED)
- ✅ Textarea interactive avec auto-save du rawContent
- ✅ Compteur de caractères
- ✅ Bouton "Analyser" actif quand texte présent
- ✅ État de chargement avec spinner (placeholder pour Epic 3)

#### Story 2.4: Brief Library & List View (COMPLETED)
- ✅ Page `/historique` avec liste des briefs
- ✅ Composant StatusBadge avec couleurs BMAD
- ✅ BriefListItem avec titre, badge, date relative, bouton supprimer
- ✅ BriefList avec tri par date et état vide
- ✅ Navigation vers l'édition au clic
- ✅ Suppression avec AlertDialog de confirmation

#### Story 2.5: Mode Switching & Brief Panel (COMPLETED)
- ✅ Composant BriefElementCard avec édition inline
- ✅ Composant BriefPanel affichant les 4 éléments BMAD
- ✅ Fonction switchMode dans BriefContext
- ✅ Mode switching via query param briefId
- ✅ Intégration du BriefPanel dans les deux modes
- ✅ Suspense boundaries pour Next.js 16

## Ressources BMAD
- Artefacts de planification: `_bmad-output/`
- Suivi workflow: `_bmad-output/bmm-workflow-status.yaml`
- Suivi sprint: `_bmad-output/implementation-artifacts/sprint-status.yaml`
- PRD complet: `_bmad-output/prd.md`
- Architecture: `_bmad-output/architecture.md`
- Spécifications UX: `_bmad-output/ux-design-specification.md`
- Epics & Stories: `_bmad-output/project-planning-artifacts/epics.md`
- Story courante: `_bmad-output/implementation-artifacts/3-1-ai-provider-abstraction-layer.md`

---
> Source: [MonprojetPro/foxeo-appli-brief](https://github.com/MonprojetPro/foxeo-appli-brief) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
