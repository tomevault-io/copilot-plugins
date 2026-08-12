## agent-cursor-supabase-cli

> L’agent Cursor PEUT pousser les migrations Supabase — ne pas renvoyer l’utilisateur sans essayer


# Agent Cursor — CLI Supabase (migrations)

## Règle impérative

- **L’agent peut et doit** exécuter les commandes Supabase CLI **depuis le dépôt** quand l’utilisateur demande de pousser des migrations ou d’appliquer le schéma distant.
- **Ne pas** répondre uniquement par « tu dois le faire toi-même » / « je ne peux pas » **sans avoir tenté** la commande ci-dessous avec permissions réseau.

## Commande à utiliser (priorité)

À la racine du projet :

```bash
npx --yes supabase db push
```

- `npx` télécharge ou utilise le CLI Supabase ; **ne dépend pas** du `supabase.exe` Windows dans le PATH.
- Nécessite que le dépôt soit **lié** au projet (`supabase/config.toml`, `project_id`), et une session **`supabase login`** valide sur la machine où tourne l’agent (souvent le même environnement que l’utilisateur).

## Fallback

- `npm run supabase -- db push` si le binaire local `tools/supabase-cli/.../supabase.exe` ou le CLI global est disponible.

## Si ça échoue

Alors seulement : indiquer l’erreur exacte et ce que l’humain peut vérifier (`supabase login`, `supabase link`, droits sur le projet `ssnqervwthlqvbewhtrd`).

## Doc projet

Voir aussi `docs/agent-supabase-cli.md` et `SETUP.md`.

---
> Source: [fsalobir/RP-Website](https://github.com/fsalobir/RP-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
