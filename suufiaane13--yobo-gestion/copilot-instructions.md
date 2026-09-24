## yobo-vitest

> Tests Vitest — lib pure, pas de Tauri


# Où tester

- **Priorité** : fonctions pures dans `src/lib/` (parse, pagination, formats, règles tailles, `userFacingError`, `toastTypeForStoreMessage`).
- **Éviter** pour l’instant : tests du store complet ou des composants sans stratégie de mock `invoke` partagée.

# Conventions

- Fichier colocalisé : `monModule.test.ts` à côté de `monModule.ts`.
- Importer explicitement depuis `vitest` : `import { describe, it, expect } from 'vitest'` (pas de globals obligatoires).
- `userFacingErrorMessage` : les assertions peuvent dépendre de `import.meta.env.PROD` (voir tests existants).

# Commandes

- `npm run test` (CI) ; `npm run test:watch` en local.

---
> Source: [suufiaane13/yobo-gestion](https://github.com/suufiaane13/yobo-gestion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
