## freelance-radar

> - Préférez l'édition (Read/Edit/Write) plutôt que Bash pour les opérations sur fichiers.

# CLAUDE.md - Instructions spécifiques au projet FreelanceRadar

## Principes de travail
- Préférez l'édition (Read/Edit/Write) plutôt que Bash pour les opérations sur fichiers.
- Pas de commentaires inutiles : ajoutez un commentaire uniquement si le pourquoi n'est pas évident.
- Pas de gestion d'erreur inutile : ne gérez que les erreurs aux limites du système (entrées utilisateur, APIs).
- Pas d'abstractions prématurées : trois lignes similaires valent mieux qu'une abstraction prématurée.
- Pas d'implémentations à moitié terminées : soit terminez une fonctionnalité, soit ne la commencez pas.

## Spécifiques au projet
- Frontend : Next.js 16 (App Router) + Tailwind CSS dans `web/`
- Backend : Bot de scraping Python dans `bot/` avec sources modulaires dans `bot/sources/`
- Base de données : Supabase avec schéma dans `supabase/migrations/`
- Sécurité : RLS activé partout, le frontend utilise la clé `anon`, le bot utilise la clé `service_role`.
- Paywall : logique centralisée dans `web/src/lib/entitlements.ts`
- Déploiement : frontend sur Vercel, bot via GitHub Actions (cron every 15 minutes).

## Commandes utiles
- Lancer le frontend : `cd web && npm run dev`
- Lancer le bot manuellement : `cd bot && python -m bot.main`
- Voir les logs du bot : vérifier la table `scrape_runs` dans Supabase.
- Déploiement bot : pousser sur GitHub, les secrets GitHub Actions sont déjà configurés.

## Points d'attention
- Les sélecteurs CSS de Codeur.com sont isolés dans `bot/config.py` pour faciliter les mises à jour.
- Le paywall freemium : annonces < 24h réservées aux Pro, résultats limités à 30 en gratuit.
- Le bot utilise l'upsert avec déduplication sur `source_url` et `content_hash` (SHA256 du contenu normalisé).

---
> Source: [eugeneglencoe-collab/freelance-radar](https://github.com/eugeneglencoe-collab/freelance-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
