## obligations-et-contexte

> Obligations et contexte pour les agents IA (lint, format, build, tests)


# Backend — Obligations et contexte pour l’agent IA

## Référence

Voir **AGENTS.md** à la racine du projet pour le contexte complet (stack, ETL, staging, commandes).

## Obligations à chaque réponse

Après toute modification de code, l’agent **doit** :

1. **Lint** : exécuter `npm run lint` et corriger les erreurs (et les warnings non voulus).
2. **Format** : s’assurer que le code est formaté (ex. `npm run format` en check ou `npm run format:write`).
3. **Build** : exécuter `npm run build` et corriger les erreurs TypeScript.
4. **Tests** : exécuter `npm run test` (ou les tests concernés) et faire en sorte qu’ils passent.
5. **Prisma** : en cas de changement de `prisma/schema.prisma`, lancer `npx prisma generate` (et migrations si besoin).

## Ordres principaux

- **Nouvelle feature / modification** : implémenter → adapter les tests → vérifier lint, format, build, tests.
- **Bugfix** : corriger → ajouter/mettre à jour un test de non-régression → vérifier lint, format, build, tests.
- **Refactoring** : ne pas changer le comportement public (ou l’adapter explicitement) → mettre à jour les tests → relancer les vérifications.
- **Schéma / ETL** : modifier le schéma si besoin → `prisma generate` (et migrations) → adapter services et mocks de tests → vérifier tout.

## Conventions

- TypeScript typé, éviter `any` sans justification.
- NestJS : controller → service → Prisma ; DTOs et interfaces dédiés.
- Tests : mocker PrismaService et HttpService ; pas de réseau/DB réels.
- Commentaires et messages en français quand c’est le standard du projet.

---
> Source: [MSPR-c-l-w/backend](https://github.com/MSPR-c-l-w/backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
