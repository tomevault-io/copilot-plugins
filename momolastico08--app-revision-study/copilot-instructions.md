## app-revision-study

> > **Nom provisoire :** StudyForge

# StudyForge — Architecture & Developer Guide

> **Nom provisoire :** StudyForge
> **But :** Application mobile de révision intelligente pour étudiants
> **Stack :** React Native + Expo (SDK 52) + TypeScript · Supabase · Anthropic API · RevenueCat

---

## Table des matières

1. [Stack technique](#stack-technique)
2. [Structure du projet](#structure-du-projet)
3. [Navigation](#navigation)
4. [Services](#services)
5. [Base de données Supabase](#base-de-données-supabase)
6. [Intégration Anthropic (IA)](#intégration-anthropic-ia)
7. [Monétisation RevenueCat](#monétisation-revenuecat)
8. [Variables d'environnement](#variables-denvironnement)
9. [Conventions de code](#conventions-de-code)
10. [Commandes utiles](#commandes-utiles)
11. [Roadmap](#roadmap)

---

## Stack technique

| Couche | Technologie | Rôle |
|---|---|---|
| Framework mobile | React Native + Expo SDK 52 | UI cross-platform (iOS / Android) |
| Langage | TypeScript (strict mode) | Typage statique |
| Navigation | Expo Router v4 (file-based) | Routing déclaratif |
| Backend / Auth | Supabase | Auth, PostgreSQL, Storage, Edge Functions |
| IA générative | Anthropic Claude (claude-opus-4-6) | Génération de fiches & quizz |
| Paiements | RevenueCat | Abonnements in-app (free / pro) |
| Style | StyleSheet natif | Pas de lib CSS-in-JS externe |

---

## Structure du projet

```
studyforge/
├── app/                        # Expo Router — routes de l'app
│   ├── _layout.tsx             # Root layout (SplashScreen, providers)
│   ├── (tabs)/                 # Groupe tabs (nav bar principale)
│   │   ├── _layout.tsx         # Configuration de la tab bar
│   │   ├── index.tsx           # Accueil
│   │   ├── library.tsx         # Bibliothèque de fiches
│   │   ├── quiz.tsx            # Quizz
│   │   └── profile.tsx         # Profil & abonnement
│   └── (auth)/                 # Groupe auth (non connecté)
│       ├── _layout.tsx
│       ├── login.tsx
│       └── register.tsx
│
├── components/                 # Composants réutilisables
│   ├── navigation/
│   │   └── TabBarIcon.tsx      # Icône Ionicons pour la tab bar
│   ├── ui/
│   │   ├── Button.tsx          # Bouton générique (primary/secondary/danger)
│   │   └── Card.tsx            # Conteneur carte avec ombre
│   ├── flashcards/
│   │   └── FlashcardItem.tsx   # Élément de liste de fiche
│   └── quiz/
│       └── QuizCard.tsx        # Carte de question QCM
│
├── screens/                    # Écrans complets (utilisés dans les routes ou modales)
│   ├── FlashcardDetailScreen.tsx
│   └── GenerateFlashcardScreen.tsx
│
├── services/                   # Couche d'accès aux données / APIs externes
│   ├── supabase.ts             # Client Supabase + helpers auth & DB
│   ├── anthropic.ts            # Appels Edge Functions pour l'IA
│   └── revenuecat.ts           # Initialisation et helpers RevenueCat
│
├── hooks/                      # Custom hooks React
│   ├── useAuth.ts              # Session Supabase (subscribe onAuthStateChange)
│   ├── useFlashcards.ts        # CRUD fiches + état loading/error
│   └── useSubscription.ts      # Tier (free/pro) via RevenueCat
│
├── types/                      # Types TypeScript globaux
│   ├── index.ts                # UserProfile, Flashcard, QuizQuestion, etc.
│   └── database.ts             # Types générés Supabase (schéma DB)
│
├── constants/
│   └── Colors.ts               # Palette de couleurs (light + dark)
│
├── assets/                     # Images, icônes, splash
│
├── supabase/                   # (À créer) Edge Functions Deno
│   └── functions/
│       ├── generate-flashcard/ # Wrapper Anthropic → fiche structurée
│       └── generate-quiz/      # Wrapper Anthropic → tableau de questions
│
├── app.json                    # Config Expo (bundle ID, scheme, plugins)
├── package.json
├── tsconfig.json               # strict + path aliases (@/*)
├── babel.config.js             # babel-preset-expo + module-resolver
└── CLAUDE.md                   # Ce fichier
```

---

## Navigation

L'app utilise **Expo Router v4** (file-based routing, similaire à Next.js).

### Groupes de routes

| Groupe | Chemin | Accès |
|---|---|---|
| `(tabs)` | `/`, `/library`, `/quiz`, `/profile` | Utilisateur connecté |
| `(auth)` | `/login`, `/register` | Utilisateur non connecté |

### Logique de redirection auth

À implémenter dans `app/_layout.tsx` avec `useAuth` :

```tsx
const { session, loading } = useAuth();

useEffect(() => {
  if (!loading) {
    if (session) router.replace('/(tabs)');
    else router.replace('/(auth)/login');
  }
}, [session, loading]);
```

### Ajout d'une route modale

Créer `app/flashcard/[id].tsx` → accessible via `router.push('/flashcard/abc123')`.

---

## Services

### `services/supabase.ts`

Client Supabase initialisé avec `expo-secure-store` pour la persistence de session native.

**Variables requises :**
- `EXPO_PUBLIC_SUPABASE_URL`
- `EXPO_PUBLIC_SUPABASE_ANON_KEY`

**Helpers exposés :**
- `signUp(email, password, firstName)` → inscription
- `signIn(email, password)` → connexion
- `signOut()` → déconnexion
- `getSession()` → session courante
- `getFlashcards(userId)` → liste des fiches
- `createFlashcard(data)` → création fiche
- `deleteFlashcard(id)` → suppression
- `getQuizSessions(userId)` → historique quizz
- `createQuizSession(data)` → nouvelle session quizz

### `services/anthropic.ts`

**IMPORTANT :** La clé Anthropic ne doit JAMAIS être exposée côté client.
Toutes les requêtes IA passent par des **Supabase Edge Functions** (serveur Deno).

**Pattern :**
```
App → Supabase Edge Function (auth token JWT) → Anthropic API
```

**Fonctions exposées :**
- `generateFlashcard(sourceText, subject, accessToken)` → `FlashcardContent`
- `generateQuiz(flashcardContent, questionCount, accessToken)` → `QuizQuestion[]`

### `services/revenuecat.ts`

Initialiser au démarrage dans `app/_layout.tsx` :

```ts
initializeRevenueCat(user?.id); // passer l'ID Supabase comme appUserID RevenueCat
```

---

## Base de données Supabase

### Migration

Fichier : `supabase/migrations/001_init.sql`
Appliquer via : `npx supabase db push`

### Tables

| Table | Rôle |
|---|---|
| `profiles` | Profil utilisateur, synchronisé depuis `auth.users` via trigger |
| `courses` | Cours regroupant des fiches (matière + niveau) |
| `flashcards` | Fiches question/réponse avec données de répétition espacée |
| `quiz_sessions` | Sessions de quizz complètes (score, durée) |
| `quiz_results` | Détail de chaque réponse dans une session |

### Schéma résumé

```
profiles       (id, email, full_name, avatar_url, tier, created_at, updated_at)
  └── courses  (id, user_id→profiles, title, subject, level, color, icon, created_at)
        └── flashcards  (id, user_id→profiles, course_id→courses,
                         question, answer, source_text,
                         difficulty 1-5, review_count, last_reviewed, next_review,
                         created_at, updated_at)
        └── quiz_sessions  (id, user_id→profiles, course_id→courses,
                            score, total_questions, duration_seconds, completed_at)
              └── quiz_results  (id, session_id→quiz_sessions, flashcard_id→flashcards,
                                 user_answer, is_correct, time_spent_seconds, created_at)
```

### RLS

RLS activé sur les 5 tables. Politique uniforme :
- `profiles` : SELECT + UPDATE sur `id = auth.uid()`
- `courses`, `flashcards`, `quiz_sessions` : ALL sur `user_id = auth.uid()`
- `quiz_results` : ALL via sous-requête sur `quiz_sessions.user_id = auth.uid()`

### Triggers

- `trg_profiles_updated_at` → met à jour `profiles.updated_at`
- `trg_flashcards_updated_at` → met à jour `flashcards.updated_at`
- `on_auth_user_created` → crée automatiquement un profil à l'inscription

### Fonctions SQL

```sql
-- Statistiques d'un cours (total fiches, dues, maîtrisées, score moyen quizz…)
select * from get_course_stats('course-uuid');

-- Fiches à réviser aujourd'hui (algorithme SRS, triées par priorité)
select * from get_due_flashcards(auth.uid(), 20);
```

### Index

- `idx_courses_user_id`
- `idx_flashcards_user_id`, `idx_flashcards_course_id`
- `idx_flashcards_next_review` (partiel : `where next_review is not null`)
- `idx_flashcards_search` (GIN trigram pour recherche plein texte)
- `idx_quiz_sessions_user_id`, `idx_quiz_sessions_course_id`, `idx_quiz_sessions_completed_at`
- `idx_quiz_results_session_id`, `idx_quiz_results_flashcard_id`

### Générateur de types

Après modification du schéma :

```bash
npx supabase gen types typescript --project-id <PROJECT_ID> > types/database.ts
```

---

## Intégration Anthropic (IA)

### Edge Functions Supabase (Deno)

Les fonctions sont dans `supabase/functions/`. Déployer avec :

```bash
npx supabase functions deploy generate-flashcard
npx supabase functions deploy generate-quiz
```

### Prompts recommandés

**generate-flashcard :**
```
Tu es un expert pédagogique. À partir du texte suivant sur "{subject}",
génère une fiche de révision structurée avec :
- Un titre clair
- Les concepts clés (liste de 5-10 points)
- Un résumé (3-5 phrases)
- Les points importants à retenir

Réponds UNIQUEMENT en JSON valide avec les clés :
{ "title": string, "key_concepts": string[], "summary": string, "key_points": string[] }
```

**generate-quiz :**
```
À partir du contenu de révision suivant, génère {n} questions QCM en français.
Chaque question doit avoir 4 options, une seule bonne réponse, et une explication.

Réponds UNIQUEMENT en JSON valide :
[{ "id": string, "question": string, "options": string[4], "correct_index": number, "explanation": string }]
```

### Modèle utilisé

`claude-opus-4-6` — le modèle le plus capable de la famille Claude 4.

---

## Monétisation RevenueCat

### Tiers

| Feature | Free | Pro |
|---|---|---|
| Fiches max | 10 | Illimité |
| Quizz / jour | 3 | Illimité |
| Générations IA / jour | 3 | Illimité |

### Initialisation

```ts
// app/_layout.tsx
import { initializeRevenueCat } from '@/services/revenuecat';

// Appeler après récupération de la session
initializeRevenueCat(session?.user.id);
```

### Hook `useSubscription`

```ts
const { tier, loading } = useSubscription();
if (tier === 'free') { /* afficher paywall */ }
```

### Identifiers produits (à configurer dans RevenueCat dashboard)

- `studyforge_pro_monthly` — abonnement mensuel
- `studyforge_pro_yearly` — abonnement annuel (recommandé)

---

## Variables d'environnement

Créer un fichier `.env` à la racine (gitignored) :

```bash
# Supabase
EXPO_PUBLIC_SUPABASE_URL=https://xxxx.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=eyJ...

# RevenueCat (préfixe EXPO_PUBLIC_ pour accès côté client)
EXPO_PUBLIC_REVENUECAT_IOS_KEY=appl_...
EXPO_PUBLIC_REVENUECAT_ANDROID_KEY=goog_...

# Anthropic (côté serveur uniquement — Edge Function Supabase)
# Ne jamais mettre dans .env côté client !
# Ajouter dans le dashboard Supabase : Settings > Edge Functions > Secrets
ANTHROPIC_API_KEY=sk-ant-...
```

---

## Conventions de code

### Fichiers

- Composants React → **PascalCase** (`FlashcardItem.tsx`)
- Hooks → **camelCase** préfixé par `use` (`useAuth.ts`)
- Services → **camelCase** (`supabase.ts`)
- Constants → **PascalCase** (`Colors.ts`)

### Imports

Utiliser les path aliases définis dans `tsconfig.json` :

```ts
import { Colors } from '@/constants/Colors';
import { Flashcard } from '@/types';
import { supabase } from '@/services/supabase';
```

### Composants

- Préférer les **function components** avec des props typées en interface
- Les styles dans un `StyleSheet.create()` en bas du fichier
- Pas de styled-components ni de Tailwind

### Gestion d'état

- État local → `useState` / `useReducer`
- État serveur → hooks custom (`useFlashcards`, `useAuth`)
- Pas de Redux ni Zustand pour l'instant (à évaluer si besoin)

---

## Commandes utiles

```bash
# Démarrer le projet
npx expo start

# iOS simulator
npx expo start --ios

# Android emulator
npx expo start --android

# Vérification TypeScript
npx tsc --noEmit

# Lint
npx eslint . --ext .ts,.tsx

# Générer les types Supabase
npx supabase gen types typescript --project-id <ID> > types/database.ts

# Déployer une Edge Function
npx supabase functions deploy <function-name>

# Build production (EAS)
npx eas build --platform ios
npx eas build --platform android
```

---

## Roadmap

### Phase 1 — MVP (en cours)

#### Acquis
- [x] Setup Expo Router + navigation tabs
- [x] Structure de dossiers et services de base
- [x] Types TypeScript et schéma DB

#### Auth & données
- [ ] Auth Supabase (login / register fonctionnels)
- [ ] CRUD fiches de révision
- [ ] Génération IA via Edge Function

#### Écrans manquants
- [ ] Onboarding : 4 écrans de présentation au premier lancement
  (valeur de l'app, fonctionnalités clés, CTA inscription)
- [ ] Écran Paramètres : changer email/mot de passe,
  supprimer compte, notifications, thème, langue
- [ ] Écran Profil complet : photo avatar, stats globales
  (fiches créées, quizz réussis, streak), badges obtenus
- [ ] Écran Paywall : présentation features Pro,
  essai gratuit 7 jours, tarifs, restore purchases

#### Quizz
- [ ] quiz.tsx : liste des cours avec nombre de fiches dispo
- [ ] QuizSessionScreen : QCM 4 choix, timer 30s par question,
  barre de progression, feedback vert/rouge immédiat,
  haptic feedback (bonne/mauvaise réponse),
  écran résultats final avec score et temps total
- [ ] Sauvegarde résultats dans quiz_sessions et quiz_results

#### Révision intelligente
- [ ] Algorithme FSRS (répétition espacée) sur les fiches :
  calcul automatique de next_review selon les performances
- [ ] Streak quotidien (jours consécutifs de révision)
- [ ] Notifications push via Expo Notifications :
  rappels personnalisés "Tu as X fiches à réviser"

#### UX/UI
- [ ] Animations et transitions entre écrans (Reanimated 2)
- [ ] Skeleton loaders sur tous les écrans de chargement
- [ ] Gestion complète des erreurs réseau avec retry
- [ ] Mode hors ligne : accès aux fiches sans internet
  (AsyncStorage local)
- [ ] Thèmes visuels : mode clair/sombre + couleur d'accent

#### Monétisation
- [ ] RevenueCat intégration complète :
  - Essai gratuit 7 jours
  - Plan Étudiant 4,99€/mois
  - Plan Pro 9,99€/mois
  - Restore purchases
- [ ] Limites appliquées côté Edge Functions :
  - Gratuit : 10 fiches/mois, 3 quizz/semaine
  - Étudiant : 50 fiches/mois, quizz illimités
  - Pro : illimité + features communauté

#### Features IA avancées
- [ ] Mode "Interroge-moi" : conversation libre avec Claude
  sur un cours, évaluation des réponses en texte libre
- [ ] Résumé automatique : génération d'un résumé 1 page
  avec points clés surlignés depuis un cours importé
- [ ] Détection des lacunes : après 5+ quizz, l'IA identifie
  les notions échouées et propose des fiches ciblées
- [ ] Planning d'examen : l'étudiant entre ses dates d'exam,
  l'app génère un planning de révision optimisé

#### Publication
- [ ] Icône app et splash screen haute résolution
- [ ] Politique de confidentialité + CGU (écran dédié)
- [ ] Support email intégré dans les paramètres
- [ ] Screenshots App Store et Google Play

---

### Phase 2 — Communauté (après lancement)

#### Banque de questions communautaire
- [ ] Table public_flashcards : fiches rendues publiques
  par les utilisateurs, filtrables par matière et niveau
- [ ] Système de votes upvote/downvote sur chaque fiche
- [ ] Modération automatique (score < -10 → masqué)
- [ ] Import de fiches publiques dans sa bibliothèque

#### Classements
- [ ] Leaderboard global par matière et niveau d'études
- [ ] Classement hebdomadaire reseté chaque lundi
- [ ] Ton rang parmi tous les étudiants de même niveau
- [ ] Médailles : or/argent/bronze top 3

#### Défi quotidien
- [ ] 10 questions communes à tous les users chaque jour
- [ ] Reset à minuit via Supabase cron Edge Function
- [ ] Classement du jour parmi tous les participants
- [ ] Streak de défis complétés

#### Système de progression
- [ ] Points XP gagnés à chaque révision et quizz
- [ ] Badges débloquables :
  "7 jours consécutifs", "100 fiches maîtrisées",
  "Top 10% de la semaine", "Première fiche publique"
- [ ] Niveau utilisateur (Débutant → Expert → Maître)

#### Mode groupe / promo
- [ ] Création de groupe avec code d'invitation
- [ ] Classement interne au groupe
- [ ] Partage de cours au sein d'un groupe
- [ ] Mode "TD en direct" : le créateur lance un quizz,
  tous les membres répondent en temps réel

---

## Limites par plan

| Feature | Gratuit | Étudiant 4,99€ | Pro 9,99€ |
|---|---|---|---|
| Fiches générées | 10/mois | 50/mois | Illimité |
| Quizz | 3/semaine | Illimité | Illimité |
| Import PDF | ❌ | ✅ | ✅ |
| Mode Interroge-moi | ❌ | ❌ | ✅ |
| Résumé automatique | ❌ | ✅ | ✅ |
| Planning d'examen | ❌ | ❌ | ✅ |
| Partage communauté | ❌ | ✅ | ✅ |
| Thèmes visuels | 1 | 3 | Tous |
| Export PDF fiches | ❌ | ✅ | ✅ |
| Mode hors ligne | ❌ | ✅ | ✅ |

---
> Source: [Momolastico08/app-revision-study](https://github.com/Momolastico08/app-revision-study) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
