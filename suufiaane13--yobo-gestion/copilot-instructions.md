## yobo-tauri-rust

> Backend Tauri / Rust — commandes, SQLite, auth


# Organisation

- **Commandes** : `src-tauri/src/commands/*.rs`, enregistrées dans `lib.rs` via `generate_handler!`.
- **DB** : `db.rs` (schéma, `migrate_schema`, `ensure_*`). Chemin fichier : éviter sous `src-tauri/` pour le watcher (déjà documenté dans le code).
- **Autorisations gérant** : `authz.rs` + vérifs dans les commandes sensibles (`ensure_active_gerant`, etc.).

# Contrats

- Réponses aux appels front : messages d’erreur en **français** clair quand c’est une erreur métier (`Result<_, String>`).
- Mots de passe : **bcrypt** ; ne pas logger les PIN.
- Erreurs internes DB : le front en **prod** masque le détail via `userFacingErrorMessage` + `client` ; côté Rust, rester cohérent (messages utilisateur vs `map_err(|e| e.to_string())` techniques).

# Style

- Garder les `command` alignées sur l’existant (serde `rename_all = "camelCase"` pour le DTO côté TS).

---
> Source: [suufiaane13/yobo-gestion](https://github.com/suufiaane13/yobo-gestion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
