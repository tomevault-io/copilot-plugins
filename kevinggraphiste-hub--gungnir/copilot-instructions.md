## gungnir

> > _Last updated : 2026-06-24, app v5.101.0_

# Gungnir — Project Guide & Conventions

> _Last updated : 2026-06-24, app v5.101.0_
> _Ce fichier est lu automatiquement par Claude Code à chaque session. Garde-le frais._

---

## Overview

Gungnir est une plateforme AI super-assistant full-stack avec architecture modulaire à plugins.

| Couche | Stack |
|---|---|
| **Backend** | FastAPI 0.115+ / Python 3.12 / SQLAlchemy 2 async / asyncpg / Postgres 16 + pgvector |
| **Frontend** | React 18.3 / TypeScript 5.6 / Vite 8 / Tailwind 3.4 / Zustand 5 / react-router 7 / i18next 24 |
| **Plugins** | 12 plugins backend (`backend/plugins/`) + frontend (`frontend/src/plugins/`) — pattern `manifest.json` + `routes.py` |
| **Infra** | Docker Compose / Nginx / runner GitHub Actions self-hosted / systemd |
| **IA / ML** | Anthropic / OpenAI / Google / OpenRouter SDKs / Ollama / `@huggingface/transformers` v3 (Whisper browser ONNX, WASM + WebGPU) |
| **Versioning** | SemVer dans `backend/core/__version__.py`, `frontend/package.json`, chaque `manifest.json` |
| **Branding** | ScarletWolf — `--scarlet` `#9B8260` (Dark Bronze par défaut), wolf identity |

---

## Quick Start

```bash
# 1. Postgres (dev)
docker compose -f compose.dev.yml up -d

# 2. Backend
export DATABASE_URL=postgresql+asyncpg://gungnir:gungnir@localhost:5432/gungnir
python -m uvicorn backend.core.main:app --host 127.0.0.1 --port 8000 --reload

# 3. Frontend
cd frontend && npm run dev
# → http://localhost:5173 (proxy /api → :8000)
```

---

## Architecture clé

- **Plugin system** : chaque plugin a un `manifest.json` (metadata + version + permissions) + `routes.py` (FastAPI). Frontend lazy-load depuis `src/plugins/`.
- **State frontend** : Zustand (`appStore`, `pluginStore`, `sidebarStore`)
- **API client** : `frontend/src/core/services/api.ts` — un seul fichier qui groupe tous les appels backend
- **Auth** : cookie HttpOnly `gungnir_session` depuis v5.0.0 (Bearer header en fallback pour scripts/tests/CI)
- **Agent tools (WOLF)** : `backend/core/agents/` — bash, filesystem, git, browser, web_fetch, etc.
- **Vite aliases** : `@` → `src/`, `@core` → `src/core/`, `@plugins` → `src/plugins/`

---

## 📋 Process & conventions globales

### Versioning & releases

- **SemVer strict** :
  - `MAJOR` = breaking change (DB migration incompatible, auth break, plugin API rewrite)
  - `MINOR` = nouvelle feature, ajout plugin, migration backward-compat
  - `PATCH` = fix, refacto sans impact utilisateur
- **Tag systématique au bump** : modifier `__version__.py` = créer immédiatement le tag `vX.Y.Z` + push. Pas optionnel.
- **CHANGELOG.md avant tag** : le push du tag déclenche le déploiement automatique VPS via `deploy-prod.yml`. Si le CHANGELOG n'est pas à jour AVANT, le release publié sera incomplet.
- **Source unique du `__version__`** : `backend/core/__version__.py` est la référence ; Vite expose `__APP_VERSION__` depuis `frontend/package.json` (à garder synchro manuellement).

### Git hygiene

- **Jamais `git add -A`** : staging explicite uniquement. `data/` Gungnir grouille de fichiers runtime untracked qu'on ne veut pas committer par accident.
- **PR systématique pour features et fix non-triviaux** : 1 PR = 1 chantier ciblé. Commit messages au format Conventional Commits-ish (`feat(scope): …`, `fix(scope): …`, `release: X.Y.Z — …`).
- **Squash merge** : on garde un historique linéaire sur `main`. `gh pr merge <N> --squash --delete-branch`.
- **Pas de force-push sur `main`**. Jamais.
- **Pré-commit pas en place** — TypeScript check (`npx tsc --noEmit`) et `python -c "import ast; ast.parse(open(f).read())"` à la main avant push pour les gros chantiers.

### Communication & process

- **Brainstorm-first** : sur tout sujet non trivial, proposer options/tradeoffs/reco AVANT toute exécution. Pas de fonçage. Cf. mémoire `brainstorm-first-then-act`.
- **Ship-then-await-feedback** : cœur livré = stop aux extras spéculatifs ; retours réels d'abord.
- **Preview HTML standalone pour UX** : quand on doit comparer 2-4 options visuelles, créer un fichier HTML autonome dans `~/` que l'user ouvre dans son browser. Évite de coder à l'aveugle et de devoir refaire.
- **Audit/clean en sub-agents parallèles** : pour les chantiers de revue (sécu, qualité), découper en 4-6 axes et lancer des sub-agents en parallèle qui écrivent leur rapport dans `audits/YYYY-MM-DD/` (gitignored).

### Sécurité (résumé des invariants)

- **Cookie HttpOnly + SameSite=Lax + Secure** pour la session — jamais de token en `localStorage` (XSS).
- **Aucun secret dans le code ni dans git** — `.env`, `.env.local`, `secrets.json` sont gitignored ; le redaction filter root masque les patterns connus dans les logs.
- **OAuth state HMAC + cookie binding** (RFC 6749 §10.12) sur tous les login flows externes.
- **`require_admin` / `require_curator` / `require_permission`** dans `auth_helpers.py` pour gater les routes sensibles.
- **`module:<plugin>` enforcement** dans `token_auth_middleware` (main.py) pour les routes `/api/plugins/<plugin>/*` quand l'user a un `role_id` assigné.
- **`_is_private_url`** (`web_fetch.py`) sur toute URL fetchée server-side — IPv4 + IPv6, fail-closed sur exception.
- **Whitelist regex stricte** sur tout identifiant utilisé en path filesystem (`plugin_id`, etc.) — pattern `^[a-z0-9][a-z0-9_.-]{0,63}$`.

### RGPD (résumé des invariants)

- **Toute donnée user-scoped** doit avoir une `user_id` (FK) — pas de leak cross-tenant.
- **Export Art. 20** : ZIP en clair (l'user doit pouvoir ouvrir).
- **Effacement Art. 17** via l'orchestrateur `rgpd_routes.py` — chemin unique avec certificat + ledger. Pas de path de delete sans audit.
- **`pii_sanitizer.py`** sur tout contenu partagé en scope collectif (Conscience).
- **`consciousness_access_logs`** append-only pour les break-glass admin.

---

## 🐍 Backend Python — bonnes pratiques

### Python 3.12

- ✅ **Type hints partout** sur signatures publiques. Style PEP 604 (`int | None`, pas `Optional[int]`).
- ✅ `from __future__ import annotations` en haut des fichiers si types complexes / forward refs.
- ✅ `match` statements OK sur Python 3.10+.
- ❌ Pas de f-string dans les logs avec données sensibles — laisser le logger gérer les args : `logger.info("X=%s", value)` permet au redaction filter de fonctionner. Pour les autres cas, f-string OK.
- ❌ Pas de `print()` dans le code runtime — toujours `logger = logging.getLogger("gungnir.<module>")`. Le redaction filter NE s'applique PAS aux prints.

### FastAPI 0.115+

- ✅ Routes async toujours (`async def`).
- ✅ `request: Request` en argument pour accéder à `request.state.user_id` posé par `token_auth_middleware`.
- ✅ `session: AsyncSession = Depends(get_session)` pour la DB.
- ✅ Réponses d'erreur : `return JSONResponse({"error": "..."}, status_code=4xx)` — cohérent avec le reste du code. `raise HTTPException` accepté pour les codes 403/404 standards.
- ✅ Routes plugin : préfixe `/api/plugins/<plugin_name>/...` via `manifest.json`. Routes core : `/api/<resource>/...`.
- ❌ Pas de logique métier dans le handler — extraire dans `services/` quand >30 lignes.
- ❌ Pas de `Depends(get_current_user)` qui throw — préférer `getattr(request.state, "user_id", None)` + check manuel pour compat open mode (≤1 user en DB).

### SQLAlchemy 2.0 (async + style legacy `Column`)

- ✅ Le projet utilise le **style `Column(...)` "legacy"** (pas `Mapped[]` / `mapped_column`). C'est cohérent partout, ne pas mixer.
- ✅ `Base = DeclarativeBase` (style 2.0) mais colonnes en style 1.x.
- ✅ `session.execute(select(X).where(...))` + `.scalars().first()` ou `.scalar_one_or_none()`.
- ✅ FK avec `ondelete="CASCADE"` quand la suppression du parent doit propager (conversations → messages, etc.).
- ✅ **Modifier un champ JSONB → `flag_modified(obj, "champ_name")`** sinon SQLAlchemy ne détecte pas le changement (Postgres only).
- ❌ Pas de `lazy="joined"` ou eager loading partout — surcharge inutile. Charger en explicite si besoin.
- ✅ **Migrations** : Alembic est le **canal officiel** depuis Phase 8b (v5.1.17). Le filet legacy (`core_migrations`, `plugins/*/migrations.py`, bloc `_has_col` dans `lifespan`) a été retiré — toutes les migrations historiques sont reflétées dans `Base.metadata` (via `Column(...)` et `__table_args__`). Le tsvector kb_chunks (Postgres pur) vit dans la revision `0002_kb_fts_setup`.
  - **Au boot** : `init_db()` fait `create_all` (couvre tout pour fresh) puis `alembic upgrade head` (idempotent, applique les revisions).
  - **Ajouter une migration** : `alembic revision -m "courte description"` à la racine repo → édite `backend/migrations/versions/<timestamp>_*.py` (functions `upgrade()` + `downgrade()`). **Idempotent obligatoire** : `IF NOT EXISTS` partout, car `upgrade head` est appelé même si déjà appliqué (pas de stamp).
  - **Tester en local** : `alembic upgrade head` avant le boot pour valider le SQL. `alembic downgrade -1` pour valider le rollback.
  - **Pas de `--autogenerate` aveugle** : review le diff manuellement avant de commit. Le générateur Alembic ne capte pas les colonnes générées Postgres (tsvector) ni les indexes partiels avec `postgresql_where`.

### asyncpg + Postgres 16 + pgvector

- ✅ DSN : `postgresql+asyncpg://...` (PAS `postgres://`).
- ✅ Postgres 16, prod a l'extension `pgvector` (compose dev a `postgres:16-alpine` simple, prod a `pgvector/pgvector:pg16`).
- ✅ Pour vector search, accès via SQL raw (pas de SDK Python pgvector dans `requirements.txt`).
- ❌ Pas de `SELECT *` raw — toujours typer via SQLAlchemy ORM ou `text()` avec colonnes nommées.

### httpx

- ✅ `httpx.AsyncClient(timeout=N)` toujours avec timeout explicite — jamais infinite.
- ✅ `follow_redirects=False` + suivi manuel avec re-check IP à chaque hop quand on fetch une URL user-supplied (anti-SSRF). Cf. `kb/extractors/url.py` et `agents/tools/web_fetch.py` pour le pattern.
- ❌ Pas de `verify=False` — jamais désactiver la vérif TLS.

### slowapi (rate-limit)

- ✅ Limiter sur les endpoints publics (pre-auth) : `/users/login`, `/users/signup`, `/users/forgot-password`, `/marketplace/submit`.
- ✅ Format `@limiter.limit("5/minute")` par IP — pour les secrets sensibles, doubler avec un rate-limit par user_id côté logique (cf. `marketplace_submission.check_rate_limit`).

### pytest

- ✅ Style fonctionnel, pas de classes. Fixtures dans `conftest.py` par dossier.
- ✅ Tests dans `backend/tests/` — un fichier par feature (`test_<feature>.py`).
- ✅ Docstring en haut du fichier qui décrit ce qui est testé.
- ❌ Pas de mocks de la DB — utiliser SQLite en mémoire ou Postgres test container. Une migration foireuse en prod doit être détectée en test.

---

## ⚛️ Frontend — bonnes pratiques

### React 18.3

- ✅ Composants fonctionnels uniquement, pas de classes.
- ✅ Hooks au top du composant, jamais dans des conditions/boucles.
- ✅ `useCallback` / `useMemo` uniquement si gain mesurable — pas de premature optimization.
- ✅ Composants exportés en `export default` (cohérent avec le plugin loader).
- ✅ Props typées via `interface Props { ... }` au-dessus du composant.
- ❌ Pas de `componentDidMount` / classe component — supprimer si rencontré.
- ❌ Pas de mutation directe de state (`state.x = ...`) — toujours via `setState(prev => ...)`.

### TypeScript 5.6

- ✅ Strict mode activé. `npx tsc --noEmit` doit passer avant tout push.
- ✅ Types partagés dans `interface` plutôt que `type` quand on veut extensibilité (sinon `type` OK).
- ✅ Préférer `unknown` à `any` quand possible. `any` toléré sur les payloads JSON externes (provider responses).
- ✅ Path aliases via `@`, `@core`, `@plugins` (cf. `vite.config.ts` + `tsconfig.json`).
- ❌ Pas de `// @ts-ignore` — préférer `// @ts-expect-error <raison>` si vraiment nécessaire.

### Vite 8

- ✅ Proxy `/api → 127.0.0.1:8000` configuré dans `vite.config.ts`. Pas de CORS en dev.
- ✅ `__APP_VERSION__` injecté à build-time depuis `package.json`. Utilisable dans le code via `define`.
- ✅ Build prod : `npm run build` génère `frontend/dist/` qui est servi par FastAPI en mode prod (StaticFiles).
- ❌ Pas d'import de fichiers depuis `backend/` côté frontend — strict séparation.

### Tailwind 3.4 + thèmes CSS variables

- ✅ Tailwind pour le layout (flex, grid, spacing, sizing). Couleurs via **CSS variables** `var(--bg-primary)`, `var(--text-secondary)`, `var(--scarlet)`, etc.
- ✅ Thèmes dans `frontend/src/core/themes/index.css` (`data-theme="dark-bronze"` est le défaut). 7 thèmes packagés + custom.
- ✅ **Appliquer un thème = `applyThemeToDOM()`** (`frontend/src/core/utils/theme.ts`) — source unique (App boot, Settings, Chat live via tool `set_theme`). Validation back dans `backend/core/services/theme.py` (whitelist 16 vars + formats sûrs, partagée tool agent ↔ API prefs). Les 3 listes (CSS `[data-theme="custom"]` / utils front / THEME_VARS back) doivent rester alignées.
- ❌ Pas de couleurs Tailwind hardcodées (`bg-red-500`, `text-gray-300`) — utiliser les variables CSS pour garder les thèmes consistants.
- ❌ Pas de `style={{ color: '#dc2626' }}` — utiliser `style={{ color: 'var(--scarlet)' }}`.

### Zustand 5

- ✅ Stores dans `frontend/src/core/stores/` — `appStore`, `pluginStore`, `sidebarStore`.
- ✅ `appStore` persiste provider/model dans localStorage (clé `gungnir_provider`, `gungnir_model`, etc.).
- ✅ Sélecteurs ciblés : `const x = useStore(s => s.x)` pour éviter les re-renders inutiles.

### react-router-dom 7

- ✅ Hash routing désactivé — on est en BrowserRouter classique.
- ✅ Routes définies dans `App.tsx`. Plugin routes ajoutées dynamiquement depuis `pluginStore`.
- ✅ Pages publiques (pre-auth) gérées en court-circuit dans `App.tsx:publicAuthRoute()` (forgot-password, reset-password, verify-email, rgpd-erasure).

### i18next 24

- ✅ Default `fr-FR`. Toutes les strings UI en français.
- ✅ Clés dans `frontend/src/i18n/locales/<lang>/<ns>.json` (un fichier par namespace, deep-merge via `import.meta.glob`).
- ✅ `useTranslation()` hook + `t('ns.key')` — tout le front externalisé (24 namespaces, FR source = 3622 clés).
- ✅ **i18n complet** : **24/24 langues officielles de l'UE à 100%** (chantier bouclé v5.69.0). Fallback `fr → en`. Jauge `frontend/scripts/i18n-coverage.mjs`. Résidu : ~60 chaînes en dur (attributs/dialogs) + quelques panneaux beta FR inline (ChatCompanionPanel, WakeWordCalibration).

### lucide-react

- ✅ Import nommé : `import { Send, Plus, X } from 'lucide-react'`. Tree-shaking efficace.
- ✅ Taille standard : `size={16}` ou `w-4 h-4` Tailwind.
- ❌ Pas d'emoji unicode dans le code (sauf cas vraiment hors UI core) — préférer lucide pour la cohérence.

### State persistance — `localStorage`

| Clé | Contenu | Sensibilité |
|---|---|---|
| `gungnir_current_user` | profil user JSON | bas (pas de token) |
| `gungnir_provider` / `gungnir_model` | LLM actif | nul |
| `gungnir_theme` / `gungnir_custom_theme` | thème UI (cache anti-flash — la **DB est la source de vérité** depuis v5.63.0 : `ui_preferences.theme`/`custom_theme_colors`, sync au boot via `useUIPreferences`) | nul |
| `gungnir_language` | i18n lang | nul |
| `gungnir_automata_view_mode` | cards/split | nul |
| `gungnir_folders_collapsed` | UI state | nul |

⚠️ **Plus jamais de token en localStorage** (cf. migration v5.0.0). Le token vit dans le cookie HttpOnly.

---

## 🐳 Infra — bonnes pratiques

### Docker Compose (dev) + multi-stage Dockerfile (prod)

- ✅ Dev : `compose.dev.yml` ne lance QUE Postgres. Backend + frontend tournent en local (uvicorn + vite).
- ✅ Prod : `deploy/Dockerfile` multi-stage (Node 20 pour build frontend, Python 3.12-slim pour runtime). Compose prod = `/opt/gungnir/docker-compose.yml` sur le VPS.
- ✅ `.env` à la racine du projet sur le VPS (chmod 600, owner github-runner, **jamais commité**).
- ❌ Pas de `latest` tag — toujours pinner les images (`postgres:16-alpine`, `node:20-slim`).

### GitHub Actions + runner self-hosted

- ✅ `deploy-prod.yml` se déclenche sur push de tag `v*.*.*` — build + deploy auto VPS avec healthcheck + rollback automatique.
- ✅ Runner self-hosted tourne sur le VPS prod directement. Pas de secrets exfiltrés via GitHub-hosted runners.
- ✅ `dependabot.yml` actif pour Python + npm.
- ❌ Pas de `secrets` GitHub utilisés pour les credentials prod (qui sont dans `/opt/gungnir/.env`).

### Nginx + systemd

- ✅ Nginx termine TLS (Let's Encrypt) et reverse-proxy vers le container `gungnir-app`.
- ✅ Header `X-Forwarded-Proto: https` posé par Nginx — lu par `_is_https()` dans `auth_cookie.py` pour le flag `Secure` du cookie.

---

## 🤖 IA / ML — bonnes pratiques

### Anthropic / OpenAI / Google / OpenRouter / MiniMax SDKs

- ✅ Provider abstraction dans `backend/core/providers/` — chaque provider implémente la même interface `BaseProvider`.
- ✅ Clés API stockées **chiffrées** en DB (`UserSettings.provider_keys` JSON, `api_key` field encrypted via Fernet `encrypt_value`).
- ✅ `get_provider(name, api_key=..., base_url=...)` factory unique dans `backend/core/providers/__init__.py`.
- ❌ Pas de SDK chargé au boot — lazy import dans chaque provider (réduit le startup time).
- ✅ **Process internes hors-HTTP** (boucles Conscience, extraction KB, reformulation HuntR…) : passer par `invoke_llm_for_user(uid, prompt, system_prompt=…)` ou, pour les tâches de fond, `invoke_background_llm(...)` (`backend/core/services/llm_invoker.py`) — ce dernier route vers le **modèle de fond** (`ui_preferences.background_llm`, réduction coûts) avec double filet (repli modèle de chat si KO + escalade si sortie invalide via `validate=`). ⚠️ `invoke_llm_for_user` n'a **pas** de `**kwargs` → on ne peut PAS passer `max_tokens`/`temperature` par cette voie.

### OpenRouter (default provider)

- ✅ Catalog public LIVE via `https://openrouter.ai/api/v1/models` — pas de snapshot statique.
- ✅ Pricing live aussi (utilisé par `CostAnalytics` pour le tracking $/user/conv).
- ✅ Inscription publique = compte non-admin **BYO-key** : l'utilisateur fournit sa propre clé (plus de clé master serveur ni de mode essai depuis v5.100.2).

### Ollama (local)

- ✅ Pas de clé API requise. Endpoint configurable via Settings.
- ✅ Provider gungnir dialogue avec l'API REST Ollama directement (`/api/chat`).
- ❌ Pas de stream tokens ralentis par latence réseau locale — assumer disponible quand activé.

### Stockage vectoriel (Conscience vs KB) — NE PAS confondre

- ⚠️ **Conscience** : le backend vectoriel est **Qdrant / Chroma / Supabase**
  (factory `core/vector/store.py`), PAS pgvector applicatif. Le `content`
  canonique vit en Postgres (`ConsciousnessMemory`, source de vérité), les
  embeddings vont dans le store configuré. Provider par défaut = `none` →
  fallback recall SQL (mots-clés + récence, depuis le plugin Conscience 4.19.0).
  L'auto-détection ne connaît que Qdrant.
- ✅ **KB** : utilise bien **pgvector** (extension `CREATE EXTENSION vector`),
  index `ivfflat` (lists=100, store partagé KB+Conscience), colonne `vector(N)`
  dans le Postgres applicatif. ⚠️ Optim recall future = passer en `hnsw` (rebuild
  d'index = chantier dédié, pas un hotfix).
- ✅ Recall scope-aware (Conscience) : fusion `private` + `collective` par
  score, puis re-rank par saillance (charge affective + leçons) en v4.19.0.

### 🧊 Conscience — GEL d'`engine.py` (refactor v5 en cours, `BRIEF_CONSCIENCE_V5.md`)

- ⚠️ **`engine.py` (~5082 l.) et `__init__.py` (~2675 l.) sont GELÉS** : d'ici la
  fin du refactor v5 en 6 Ports, **aucune nouvelle logique** n'entre dans ces deux
  fichiers. Tout nouveau comportement de Conscience naît dans un **fichier séparé**.
- 🔒 Gravé mécaniquement : `backend/tests/test_consciousness_freeze.py` (cliquet
  anti-croissance) **échoue en CI** si l'un des deux dépasse son plafond. Le
  refactor strangler ne fait que RÉDUIRE ces fichiers → **abaisser le plafond**
  dans ce test à chaque port migré (cliquet descendant).
- 💾 Avant tout refactor qui touche l'état : `python3 scripts/conscience_snapshot.py`
  (sauvegarde restaurable + vérifiable de `data/consciousness/` + tables canoniques
  Postgres). L'état accumulé (héritage Hugginn) est plus précieux que le code.
- 🗺️ Carte d'extraction turnkey (Ports → symboles/lignes réels) :
  `CONSCIENCE_V5_REFACTOR_PLAN.md`. Brief source : `BRIEF_CONSCIENCE_V5.md`.

### `@huggingface/transformers` v3 (Whisper browser)

- ✅ Modèle Whisper tourne **dans le browser** via ONNX Runtime Web. Pas de serveur.
- ✅ Migré de `@xenova/transformers` v2 → `@huggingface/transformers` v3 (v5.17.0) pour débloquer **WebGPU** (palier dictée « Turbo » = `whisper-large-v3-turbo`). Au passage, la CVE protobufjs transitive de la v2 disparaît.
- ✅ Paliers de qualité dans `whisper.ts` (`STT_TIERS`) : `rapide`=base/WASM, `precis`=small/WASM (défaut), `turbo`=large-v3-turbo/WebGPU. Le worker (`whisper.worker.ts`) reçoit `model`/`device`/`dtype` et **retombe sur small WASM** si WebGPU/turbo échoue (fallback gracieux). `currentSttTier()` garde-fou : turbo sans WebGPU → precis (pas de download inutile).
- ✅ Web Worker dédié (`whisper.worker.ts`) pour ne pas freezer l'UI.
- ✅ Params anti-hallucination/boucle réglés (cf. comment dans le worker — héritage de tuning Kevin).
- ❌ Décodage `data:` URL **sans** `fetch()` — utiliser `atob` direct, sinon CSP bloque.

### Prompts agent (WOLF system)

- ✅ Prompts dans `backend/core/agents/system_prompts/` + per-user `soul.md` overlays.
- ✅ Outils déclarés dans `wolf_tools.py` (schémas + executors) + auto-discovery MCP.
- ✅ Tool-gating UNIQUEMENT sur les outils MCP (cf. v4.27.2). Les outils natifs WOLF sont toujours envoyés au modèle pour fiabilité.

---

## 🔗 Compatibilité des technos

Matrice des versions actuelles + contraintes croisées. À **vérifier avant tout
bump majeur** d'une dépendance — un upgrade isolé peut casser une autre brique.

### Stack Backend (Python)

| Techno | Version actuelle | Compatible avec | Pièges connus |
|---|---|---|---|
| **Python** | 3.12 (prod), 3.10+ (min) | tout le reste | 3.13 pas encore testé ; 3.14 bloque pip-audit sur certaines machines (cf. ops VPS) |
| **FastAPI** | ≥ 0.115 | Pydantic v2 OBLIGATOIRE, Starlette ≥ 0.40, uvicorn ≥ 0.30 | FastAPI < 0.100 = Pydantic v1 only (incompat majeure) |
| **Pydantic** | v2 (≥ 2.9) | FastAPI ≥ 0.100, SQLAlchemy 2 | Migrer depuis v1 demande de refaire les `Config` → `model_config`, `validator` → `field_validator`. Tout doit être migré d'un coup |
| **SQLAlchemy** | 2.0 async (`[asyncio]`) | asyncpg, Pydantic v2, alembic ≥ 1.14 | Style **`Column(...)` legacy** utilisé partout (pas `mapped_column`). NE PAS mixer — tout migrer en une fois ou rien |
| **asyncpg** | ≥ 0.30 | Postgres 12+ (Postgres 16 chez nous), SQLAlchemy `[asyncio]` | DSN doit être `postgresql+asyncpg://` (PAS `postgres://`) |
| **Postgres** | 16 (dev + prod) | pgvector (prod), asyncpg, alembic | Dev = `postgres:16-alpine` simple ; **prod = `pgvector/pgvector:pg16`** (extension activée). À garder synchro |
| **alembic** | ≥ 1.14 | SQLAlchemy 2 | Installé mais **pas encore branché** — migrations vivent dans `main.py:lifespan` actuellement |
| **httpx** | ≥ 0.27 | Python 3.10+, HTTP/2 via `h2` | Pas de remplacement par `requests` (sync only) |
| **aiohttp** | ≥ 3.10 | Python 3.10+ | Utilisé par `web_fetch` pour la garde SSRF (suit pas-à-pas les redirects, à l'inverse de httpx classique) |
| **websockets** | ≥ 13 | uvicorn `[standard]`, FastAPI | Pour le voice realtime et le LSP du plugin code |
| **anthropic** | ≥ 0.37 | Python 3.10+ | Claude Messages API natif |
| **openai** | ≥ 1.50 | Python 3.10+ | SDK v1 (async client) |
| **google-generativeai** | ≥ 0.8 | Python 3.10+ | Note : Google a aussi sorti `google-genai` (deprecation potentielle) |
| **slowapi** | latest | FastAPI / Starlette | Rate-limit par IP via décorateur |
| **cryptography** | ≥ 43 | Python 3.10+ | Fernet (encrypt_value), Ed25519 (signature plugins) |
| **pytest** | latest | Python 3.10+ | Tests dans `backend/tests/`. Pas de mocks DB |

### Stack Frontend (Node)

| Techno | Version actuelle | Compatible avec | Pièges connus |
|---|---|---|---|
| **Node** | 20-slim (build prod), ≥ 18 (min dev) | npm, vite, react | Migration vers 22 LTS non testée |
| **TypeScript** | ~5.6.2 | React 18, Vite 8, types officiels | `strict: true` activé ; pas de `// @ts-ignore`, préférer `@ts-expect-error` |
| **React** | 18.3.1 | TS 5.6, react-dom 18.3, react-router 7 | React 19 = breaking (use, ref-as-prop) — pas avant migration explicite |
| **react-dom** | 18.3.5 | React 18.3 | Pas de mismatch entre react et react-dom (faux positifs de hooks) |
| **Vite** | ^8.0.13 | React 18+, TS 5+, Node 18+ | Vite 8 est récent (2025) — `@vitejs/plugin-react` ^6 obligatoire |
| **react-router-dom** | 7.x | React 18+ | v7 = breaking par rapport à v6 (loaders, actions) — tout est en v7 native, ne pas downgrade |
| **Tailwind** | 3.4.x | PostCSS 8, autoprefixer 10 | Tailwind v4 = breaking (sortie 2024) — pas encore migré |
| **Zustand** | 5.x | React 18+ | v5 = breaking par rapport à v4 (suppr getServerSnapshot) |
| **i18next + react-i18next** | 24.x / 15.x | React 18+ | 24/24 langues UE à 100% (FR source 3622 clés) |
| **lucide-react** | ^0.468 | React 18+ | Import nommé pour le tree-shaking |
| **@huggingface/transformers** | ^3.8 | Node 18+ | Bundle ONNX Runtime Web (WASM + **WebGPU**). Worker module-only. A remplacé `@xenova/transformers` v2 en v5.17.0 (débloque WebGPU + résout la CVE protobufjs de la v2) |
| **@xyflow/react** | ^12 | React 18+ | Forge canvas (workflows) |
| **@codemirror/*** | ^6 | React via `@uiw/react-codemirror` | Plugin code |
| **cytoscape + dagre** | ^3 / ^2 | navigateur moderne | Conscience graphe |

### Stack Infra

| Techno | Version actuelle | Compatible avec | Pièges connus |
|---|---|---|---|
| **Docker** | engine moderne (Compose v2) | tout | `docker compose` (sans tiret) — la syntaxe v1 `docker-compose` est dépréciée |
| **Docker Compose** | v2 (déclaratif YAML) | Docker engine | Pin les images (`postgres:16-alpine`, jamais `latest`) |
| **Nginx** | stable upstream | TLS 1.2+ (Let's Encrypt) | Header `X-Forwarded-Proto` doit être posé, lu par `auth_cookie._is_https` |
| **runner GH self-hosted** | latest | GitHub Actions | Sur le VPS prod ; secrets jamais exposés à GitHub-hosted runners |

### Croisements critiques à connaître

- **FastAPI / Pydantic / SQLAlchemy** : trio verrouillé en version major. Bumper FastAPI vers une major suivante peut imposer un bump Pydantic ou SQLAlchemy — vérifier la matrice officielle FastAPI avant.
- **asyncpg + Postgres 16** : OK. Si on passe à Postgres 17, vérifier que `pgvector` a une release pour 17 (peut traîner).
- **React 18 + react-router 7** : OK actuellement. React 19 + react-router 7 = OK aussi (selon docs RR), mais on n'y est pas.
- **Vite 8 + plugin-react 6** : pinning serré, ne pas désynchroniser. Vite 7 ↔ plugin-react 5, Vite 8 ↔ plugin-react 6.
- **Tailwind 3 → 4** : breaking sur la config + classes. À planifier en chantier dédié.
- **`@huggingface/transformers` v3 (WebGPU)** : worker Whisper en module worker. Le palier turbo exige WebGPU (`navigator.gpu`) ; fallback WASM small géré dans `whisper.worker.ts`. La CVE protobufjs de l'ancienne v2 `@xenova/transformers` a disparu avec la migration (v5.17.0).
- **`google-generativeai` ↔ `google-genai`** : Google pousse vers le nouveau package `google-genai` (officiel) et déprécie l'ancien. À migrer dans un chantier dédié quand un blocker apparaît.
- **Postgres dev ≠ prod** : `compose.dev.yml` utilise `postgres:16-alpine` (pas d'extension pgvector). Si on développe sur du code qui requiert pgvector, basculer dev sur `pgvector/pgvector:pg16` aussi (modifier compose).
- **Python 3.12 vs 3.14** : prod Docker = 3.12. Si dev sur 3.14 (Linux récent), certains outils (`pip-audit`, `bandit`) peuvent ne pas être dispo (cf. audit).

### Process bump deps

1. **Backend** : modifier `backend/requirements.txt` + `deploy/requirements.txt` ensemble. `pip install -r backend/requirements.txt` localement pour vérifier.
2. **Frontend** : `npm install <pkg>@<v>` puis `npx tsc --noEmit` + `npm run build` pour catcher les régressions de types.
3. **Vérifier les transitifs** : `npm audit` + `pip-audit` (ou `pip list --outdated`).
4. **Pour les CVE actives** : prioritiser le bump. Cf. audit `04-secrets-crypto-supply.md` pour les CVE connues.

---

## 🛡️ Sécurité — invariants à respecter

Cf. aussi audits `audits/2026-05-26/` (gitignored, local-only).

| Invariant | Comment c'est appliqué |
|---|---|
| Token jamais en localStorage | Cookie HttpOnly + SameSite=Lax + Secure (v5.0.0) |
| OAuth state non forgeable | HMAC fallback aléatoire au boot + cookie binding (RFC 6749 §10.12) |
| Account-link OAuth non-hijackable | `email_verified=True` requis sur le compte existant (anti Microsoft 2022) |
| RBAC backend appliqué | Middleware `module:<plugin>` check pour les routes plugins |
| SSRF anti-IPv6 / IPv4-mapped | `_is_private_url` + `getaddrinfo(AF_UNSPEC)` (v5.0.7) |
| Plugins admin-only | `require_admin` sur install/uninstall marketplace |
| IDOR évité | `enforce_conversation_owner` sur toute route conv-scoped |
| Path traversal bloqué | Whitelist regex `^[a-z0-9][a-z0-9_.-]{0,63}$` sur `plugin_id` ; basename + confinement sur les écritures fichier (download navigateur) |
| Logs sans secrets | Redaction filter root + `logger.*` partout (pas de `print()`) |
| XSS — `href` rendus | `isSafeUrl`/`safeHref` (`core/utils/url.ts`) sur toute URL web/LLM mise en `href` (React + exports). `javascript:`/`data:` rejetés (v5.2.0) |
| Webhooks signés fail-closed | Discord Ed25519 (PyNaCl) **refuse** si non vérifiable ; jamais « valide par défaut » (v5.2.0) |
| SSRF crawler | `_is_private_url` sur chaque URL + suivi manuel des redirects (`web_crawl_lite`, v5.2.0) |
| Anti-bombe de décompression | Cap par membre + ratio + décompression bornée par chunks sur les archives KB (v5.2.0) |
| Caviardage scope collectif (KB) | `sanitize_for_collective` (secrets + PII) à l'ingestion d'une collection collective, avant chunking/embed (v5.2.0) |
| Admin = vrai `is_admin` | `require_admin`/`require_curator` ; jamais de check `user_id == 1` en dur |
| Re-auth opérations sensibles | Mot de passe actuel exigé au changement d'email (hors OAuth-only, v5.2.0) |
| Anti-lockout RGPD | Effacement du dernier administrateur actif refusé (v5.2.0) |
| Export CSV | Neutralisation de l'injection de formule (`= + - @`) (v5.2.0) |

**Chantiers sécu — suite du plan post-audit 2026-05-29** (rapports
`audits/2026-05-29/` local-only). Phase 1 (durcissement A + isolation
multi-tenant B) **livrée en v5.2.0**. Restent, par ordre :
- **Lot C — keystone RBAC** : brancher réellement `require_permission`
  (catalogue `cap:*` aujourd'hui décoratif, jamais appelé) + feature
  « restriction de modèles LLM par groupe/user » via le panneau admin.
- **Lot D — policies admin-configurables** : brancher le rate-limit global
  (`SlowAPIMiddleware` jamais ajouté) mais **réglable par rôle**, quotas
  (image/STT/ingest) par groupe.
- **Lot E — isolation de l'agent** (archi, pas patch) : secret brokering
  (clé maître hors de portée de l'exec agent) + trust-tainting (web/KB
  marqués « non fiable » → gating des outils puissants = vraie réponse à la
  prompt injection). **Ne PAS brider** l'agent auto-améliorable.
- Impersonation refonte avec table dédiée + token séparé (avant 2e admin)
- SSRF anti-DNS-rebinding (custom transport httpx + IP pinning) avant migration AWS/GCP
- KDF sous OWASP : PBKDF2 100k→600k ou Argon2id (migration douce au login)

---

## 📦 Plugin system — bonnes pratiques

### Backend (`backend/plugins/<name>/`)

```
<name>/
  manifest.json    # metadata + version + permissions + sidebar_position
  routes.py        # FastAPI routes (mounted at /api/plugins/<name>/)
  __init__.py      # optional lifecycle hooks
  ...              # libre
```

- ✅ Versionner indépendamment du core (le plugin a sa propre `version` dans manifest).
- ✅ `manifest.json` champs obligatoires : `name`, `version`, `display_name`, `route`, `icon` (nom lucide), `sidebar_position`, `core_required` (bool).
- ✅ Si le plugin manipule des données user-scoped, **toujours filtrer par `request.state.user_id`** — pas de SELECT sans WHERE user_id.
- ✅ Si le plugin expose des actions sensibles, ajouter à `permissions.py` la `cap:` correspondante et gater via `require_permission`.

### Frontend (`frontend/src/plugins/<name>/`)

```
<name>/
  manifest.json    # mirror du backend manifest (display_name, icon, route)
  index.tsx        # export default ComponentName
  ...              # composants locaux
```

- ✅ Composant principal `export default` (le plugin loader cherche ce default).
- ✅ Pas de dep core par défaut, mais OK d'importer `@core/components/*` pour UI partagée (PageHeader, TabBar, PrimaryButton…).
- ✅ Plugin doit fonctionner standalone — pas de fetch silencieux vers d'autres plugins sans nécessité.

---

## 🎨 UX / a11y — bonnes pratiques

Cf. audit `06-ux-ui-a11y.md` pour le détail des trous restants.

- ✅ **Couleurs via variables CSS**, jamais hardcodées
- ✅ **Tailwind `focus-visible`** pour les états focus clavier
- ✅ **`touch-target` ≥ 44px** sur mobile (boutons icon-only avec padding)
- ✅ **Reduced-motion respecté** (cf. `themes/index.css`)
- ⚠️ **Modales** : pattern `<dialog>` à harmoniser — actuellement mix `fixed inset-0` + portail. Composant `Modal` partagé existe dans `plugins/kb/components/Modal.tsx`.
- ⚠️ **Formulaires** : majorité des `<label>` ne sont pas liés via `htmlFor` au champ — à corriger progressivement.
- ✅ **i18n** : front externalisé, 24/24 langues UE à 100% (cf. plus haut) ; résidu = ~60 chaînes en dur + panneaux beta FR inline.
- ❌ Plus de `confirm()` / `alert()` natifs (audit en a recensé ~103) — préférer un composant modal styled cohérent quand on touche un fichier qui en utilise.

---

## 🔧 Outils Claude Code dans ce projet

Quelques skills à connaître quand tu bosses sur Gungnir :

| Skill | Usage |
|---|---|
| `/ultrareview <PR#>` | Multi-agent cloud review d'une PR. **3 free / mois**, payant ensuite. À garder pour les PRs sécu/touchy. |
| `/security-review` | Review sécu locale des changements pending. Léger, gratuit. |
| `/init` | Initialise un CLAUDE.md (déjà fait, pas à ré-exécuter) |
| `/run` | Lance l'app et observe le runtime |
| `/verify` | Vérifie qu'un change marche en réel |

---

## 📜 Process release Gungnir (à reproduire)

```bash
# 1. Bump version
sed -i 's/__version__ = ".*"/__version__ = "X.Y.Z"/' backend/core/__version__.py
# + frontend/package.json + frontend/package-lock.json (npm install) + badge README

# 2. RESYNC DOC (sinon dérive — cf. incident ABOUT bloqué à v3.42.1) :
#    en-tête ABOUT.md (version + date), CLAUDE.md (ligne 3), GUIDE.md (version),
#    + CHANGELOG.md (nouvelle section EN HAUT).
bash scripts/check-docs-version.sh   # DOIT passer avant de tagger (garde-fou)

# 3. Commit + tag
git add CHANGELOG.md backend/core/__version__.py frontend/package.json frontend/package-lock.json README.md ABOUT.md CLAUDE.md GUIDE.md
git commit -m "release: X.Y.Z — <résumé court>"
git tag vX.Y.Z

# 4. Push (déclenche deploy auto VPS)
git push origin main
git push origin vX.Y.Z
```

> 🛡️ Un hook local (`scripts/hooks/pretag-doc-guard.sh`, à brancher dans
> `~/.claude/settings.json`) bloque `git tag vX` si ABOUT/CLAUDE/GUIDE ne portent
> pas la version courante. Le vérificateur `scripts/check-docs-version.sh` est
> réutilisable en CI.

⚠️ **Ne JAMAIS** :
- Push un tag sans avoir update CHANGELOG.md **ET resync ABOUT/CLAUDE/GUIDE** (version + date)
- Bump version sans tag dans la foulée
- Force push sur `main`
- Skipper `--no-verify` sur les hooks (il n'y en a pas activé, mais si on en ajoute)

---

## 📚 Pour aller plus loin

- `CHANGELOG.md` — historique complet par version
- `RGPD.md` / `RGPD_POLITIQUE.md` — chantier conformité
- `audits/2026-05-26/` (gitignored, **local seulement**) — 6 rapports d'audit sécu/qualité
- `~/.claude/projects/-home-scarletwolf-kevin/memory/` — mémoires persistantes Claude Code

---
> Source: [kevinggraphiste-hub/Gungnir](https://github.com/kevinggraphiste-hub/Gungnir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
