## backend

> Ce document décrit le contexte du projet et les **obligations** que tout agent IA doit respecter lors des modifications de code.

# Guide pour les agents IA — Backend

Ce document décrit le contexte du projet et les **obligations** que tout agent IA doit respecter lors des modifications de code.

---

## 1. Contexte du projet

### 1.1 Stack technique

- **Framework** : NestJS (Node.js)
- **Langage** : TypeScript
- **Base de données** : MariaDB via **Prisma** (client + migrations)
- **Outils** : ESLint (type-checked), Prettier, Jest

Un **back-office** (Next.js) consomme cette API pour le dashboard, les logs ETL et l’administration.

### 1.2 Structure principale

- `src/` : code source (modules NestJS par domaine)
- `prisma/` : schéma Prisma (`schema.prisma`), migrations, seed
- Modules métier : `auth`, `users`, `organization`, `roles`, `health-profile`, `nutrition`, `exercice`, `session`, `session-exercise`, `plan`, `subscription`, `prisma`

### 1.3 ETL et tables Staging

Le backend expose des **pipelines ETL** qui n’écrivent **pas** directement dans les tables métier. Les données nettoyées sont enregistrées dans des **tables de staging** pour validation humaine avant insertion en base finale.

- **Tables staging** : `NutritionStaging`, `ExerciseStaging`, `HealthProfileStaging`
- **Champs** : `id` (UUID), `cleaned_data` (JSON), `anomalies` (JSON), `status` (PENDING | APPROVED | REJECTED), `created_at`, `updated_at`
- **Services concernés** :
  - `NutritionService.runImportPipeline()` → écrit dans `NutritionStaging`
  - `ExerciceService.runImportPipeline()` → écrit dans `ExerciseStaging`
  - `HealthProfileService.runHealthProfilePipeline()` → écrit dans `HealthProfileStaging`

Les tables finales (`Nutrition`, `Exercise`, `HealthProfile`) sont alimentées après revue (hors scope des pipelines ETL actuels).

### 1.4 Commandes utiles

| Commande | Rôle |
|----------|------|
| `npm run build` | Compilation TypeScript |
| `npm run lint` | Vérification ESLint |
| `npm run lint:fix` | Correction automatique ESLint |
| `npm run format` | Vérification Prettier (check only) |
| `npm run format:write` | Application Prettier |
| `npm run test` | Tests unitaires Jest |
| `npx prisma generate` | Régénération du client Prisma après changement de schéma |

---

## 2. Obligations à chaque requête

**L’agent doit systématiquement :**

1. **Vérifier le lint**  
   Après toute modification de code, exécuter `npm run lint` (ou utiliser l’outil de lint du projet). Corriger toute erreur ou warning pertinent (les warnings peuvent être acceptés si justifiés).

2. **Vérifier le format**  
   S’assurer que le code respecte Prettier : `npm run format` (check) ou `npm run format:write` (appliquer). Aucun changement de format non intentionnel.

3. **Vérifier le build**  
   Après modifications dans `src/` ou `prisma/`, lancer `npm run build` et corriger les erreurs TypeScript.

4. **Vérifier les tests**  
   Si des fichiers `.spec.ts` existent ou ont été modifiés, lancer `npm run test` (ou les tests concernés). Les tests doivent rester verts ; adapter les tests si le comportement attendu change.

5. **Prisma**  
   En cas de modification de `prisma/schema.prisma`, exécuter `npx prisma generate` et créer/appliquer les migrations si nécessaire.

---

## 3. Ordres principaux par type de requête

### 3.1 Nouvelle fonctionnalité ou modification de code

1. Implémenter la demande en respectant l’architecture existante (modules, services, DTOs, interfaces).
2. Adapter ou ajouter les tests unitaires concernés.
3. Exécuter : `npm run lint` → `npm run format` (ou `format:write`) → `npm run build` → `npm run test`.
4. Corriger jusqu’à ce que lint, format, build et tests soient OK.

### 3.2 Correction de bug

1. Identifier et corriger la cause.
2. Ajouter ou modifier un test qui couvre le cas (régression).
3. Vérifier lint, format, build et tests comme en 3.1.

### 3.3 Refactoring

1. Effectuer le refactoring sans changer le comportement public (ou en l’adaptant de façon explicite).
2. Mettre à jour les tests existants si les signatures ou les contrats changent.
3. Relancer lint, format, build et tests.

### 3.4 Modification du schéma Prisma / ETL

1. Modifier `prisma/schema.prisma` si besoin.
2. Lancer `npx prisma generate` ; créer une migration si des changements de schéma DB sont requis.
3. Adapter les services (et les mocks dans les specs) qui utilisent les modèles modifiés (ex. staging).
4. Vérifier lint, format, build et tests.

---

## 4. Conventions de code

- **TypeScript** : typage explicite quand cela améliore la lisibilité ou la sécurité ; éviter `any` sauf si nécessaire (avec commentaire ou eslint-disable ciblé).
- **NestJS** : respecter le découpage controller → service → Prisma ; DTOs et interfaces dans les dossiers dédiés.
- **Tests** : mocker `PrismaService` (et `HttpService` pour les pipelines ETL) ; pas d’appel réseau ou DB réels dans les tests unitaires.
- **Français** : commentaires et messages utilisateur en français lorsque c’est le standard du projet.

---

## 5. Résumé checklist (à appliquer en fin de réponse)

- [ ] `npm run lint` sans erreur (warnings explicites si conservés)
- [ ] `npm run format` OK ou `npm run format:write` exécuté
- [ ] `npm run build` OK
- [ ] `npm run test` OK (ou tests ciblés passants)
- [ ] Si schéma Prisma modifié : `npx prisma generate` (et migrations si besoin)

---
> Source: [MSPR-c-l-w/backend](https://github.com/MSPR-c-l-w/backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
