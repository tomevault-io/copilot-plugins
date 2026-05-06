## vtc-memory-token-min

> VTC HUB: contexte + protocole token-min (sources locales)

<!-- /home/mike/projects/vtc/vtc_mvp/.cursor/rules/vtc-memory-token-min.mdc -->

## Source of truth (local-first)
- `.ai_memory/HISTORY.md` + `.ai_memory/PLAN_ARCHITECTURE.md` : **où on en est**
- `docs/flows.md` + `docs/architecture.md` : **comment ça marche**
- `docs/database.md` + `docs/security.md` + `docs/decisions.md` : **contraintes**
- `.agent/instructions.md` + `.agent/rules.md` + `.agent/identity.md` : **style/protocol**

## Non-négociables (projet)
- **Stack**: Astro SSR + Supabase (Postgres/Auth/RLS) + Stripe Connect + Cloudflare.
- **Multi-tenant**: `tenant_id` partout, **RLS** activé, pas d’accès cross-tenant.
- **Stripe**: idempotence via `stripe_events`, ledger via `financial_movements` (immuable).
- **Repo rule**: ce repo = backoffice/ERP (donnée + pricing + ledger). Le public ne calcule pas les prix.
- **Front priority**: **Mobile First** (la version mobile prime sur desktop).

## Token minimization (hard)
- Lire **1–2 fichiers max** avant de répondre.
- Préférer `Grep` ciblé (string/symbol) à la lecture complète.
- Interdiction d’utiliser les gros digests (`repo_context.xml`, `repomix-output.md`) sauf demande explicite.
- Si un point est ambigu: **question courte** plutôt qu’inventer.

## Sécurité
- **Jamais** de secrets en dur. Toujours `os.getenv` / variables d’environnement.
- **Ne pas utiliser** de MCP Supabase “direct DB”. Préférer REST/JSON et clés `.env` (selon les règles `.agent/*`).

## Tri-mémoire (process)
- Après chaque changement:
  - `.ai_memory/HISTORY.md` : 1 bullet (fait, daté)
  - `.ai_memory/EVOLUTION.md` : 1 bullet (next si besoin)
  - `.ai_memory/PLAN_ARCHITECTURE.md` : seulement si la roadmap change

## Sortie (format)
**Statut** : Succès/Erreur  
**Action** : résumé chirurgical (1 ligne)  
**Next** : prochaine étape (1 ligne)

---
> Source: [mike-banner/VTC_MVP](https://github.com/mike-banner/VTC_MVP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
