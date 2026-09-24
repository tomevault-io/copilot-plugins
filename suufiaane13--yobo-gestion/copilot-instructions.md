## yobo-project-core

> YOBO Gestion — stack, français, données, CI (toujours actif)


# Contexte

Application **bureau Tauri 2** + **React 19** + **TypeScript** + **Vite 8** + **Tailwind 4** + **Zustand 5**. Métier : **caisse / menu / utilisateurs / historique** (snack). Toute l’UI et les **messages utilisateur** sont en **français**.

## Données

- SQLite via Rust ; fichier sous le **dossier données local** de l’OS (`db.rs` / `open_db_file`), pas dans le dépôt.
- Ne jamais versionner `src-tauri/target/`, `dist/`, `node_modules/`.

## Qualité attendue

- `npm run lint`, `npm run test`, `npm run build` doivent passer après un changement non trivial.
- CI : `.github/workflows/ci.yml` (lint + tests + build front). Le build **Tauri complet** reste local sauf ajout explicite.

## Portée des changements

- Modifier **uniquement** ce qui est demandé ; pas de refactor large du store ou des pages « par habitude ».

---
> Source: [suufiaane13/yobo-gestion](https://github.com/suufiaane13/yobo-gestion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
