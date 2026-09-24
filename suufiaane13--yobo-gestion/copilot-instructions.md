## yobo-frontend-react

> Front React/Zustand — textes client, invoke, effets


# Structure

- **Pages** : `src/pages/*` ; **composants** : `src/components/*`.
- **État** : `src/store/yobo-store.ts`, état initial `yobo-store-state.ts`, effets `yobo-store-effects.tsx`.
- **Types** : domaines dans `src/types/*.ts` ; agrégation via `yoboApp.ts` quand c’est déjà le pattern.

# Textes utilisateur

- Libellés succès / erreur / validation : **`src/lib/yoboClientMessages.ts`** (`client.*`). Éviter les longues chaînes en dur dans les composants.
- Toasts : types `error` | `warning` | `success` ; classification des messages globaux via `toastTypeForStoreMessage` quand ils passent par `setError` + effet.

# Tauri `invoke`

- Dans les `catch` : `logDevError('nom_commande', e)` + `userFacingErrorMessage(e, client.error.…)` (ou `setXxxError` avec le même fallback).
- Ne pas afficher de messages techniques bruts en prod (déjà géré dans `userFacingError.ts` si le fallback vient de `client`).

# React

- Préférer les hooks existants (`useShallow` Zustand quand plusieurs champs). Pas de nouveau routeur : navigation par **onglet** dans `App.tsx`.

---
> Source: [suufiaane13/yobo-gestion](https://github.com/suufiaane13/yobo-gestion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
