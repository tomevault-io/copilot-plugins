## appfood

> Application mobile de suivi nutritionnel personnalise pour vegans/vegetariens sportifs.

# appFood — Instructions pour Claude Code

Application mobile de suivi nutritionnel personnalise pour vegans/vegetariens sportifs.
Stack : Kotlin Multiplatform + Compose Multiplatform (mobile) + Ktor (backend) + PostgreSQL + Meilisearch.

## Documents de reference

| Document | Contenu |
|----------|---------|
| `CONVENTIONS.md` | Conventions de code, architecture, patterns obligatoires |
| `docs/phase1-vision-rapport.md` | Vision produit, personas, modele economique |
| `docs/phase2-architecture-rapport.md` | Architecture technique, stack, infrastructure |
| `docs/phase3-backlog-rapport.md` | Backlog complet avec toutes les User Stories |
| `docs/phase4-dispatch-plan-agents.md` | Plan d'execution des agents, sprints, parallelisme |
| `docs/project-structure.md` | Arborescence du projet, mapping Agent → Fichiers |
| `docs/data-models.md` | Modeles de donnees (Kotlin, Exposed, SQLDelight) |
| `docs/api-contracts.md` | 50 endpoints API avec Request/Response DTOs |
| `docs/us-clarifications.md` | Specifications detaillees pour QUOTAS-01, RECO-01/02, DATA-01, SYNC-01/02 |
| `docs/sprint-tracker.md` | Suivi d'avancement des sprints |
| `docs/TODO-HUMAIN.md` | Actions humaines requises (cles API, comptes, contenu) |
| `docs/WAITING-REVIEW.md` | US en attente de validation utilisateur |

## Agents

7 agents specialises, orchestres par PROJECT-MASTER :
- **SHARED** — Logique metier KMP (`shared/` sauf `ui/`)
- **MOBILE** — UI Compose Multiplatform (`shared/ui/`)
- **BACKEND** — API Ktor (`backend/`)
- **DATA** — Import Ciqual, AJR, Meilisearch
- **INFRA** — CI/CD, Docker, Railway
- **REVIEW** — Revue de code (lecture seule)
- **PROJECT-MASTER** — Orchestrateur

Skills agents : `.CLAUDE/commands/agent-*.md`

## Infrastructure

| Service | URL |
|---------|-----|
| Backend (production) | `https://appfood-production.up.railway.app` |
| Health check | `GET /api/health` |
| PostgreSQL (interne) | `postgres.railway.internal:5432/railway` |
| Meilisearch (interne) | `http://meilisearch.railway.internal:7700` |
| Firebase projet | `foodapp-5ea23` |

Backend en mode `FIREBASE_MOCK=true` (pas encore de verification Firebase en prod).

## Regles cles

- **Architecture** : Clean Architecture (UseCase → Repository → DataSource) cote shared, Routes → Service → DAO cote backend
- **Erreurs** : `AppResult<T>` partout dans shared, jamais de throw. Exceptions metier dans backend (StatusPages les intercepte)
- **Serialization** : kotlinx.serialization uniquement, pas de Gson/Moshi/Jackson
- **DI** : Koin uniquement, pas de Dagger/Hilt
- **UI** : Compose Multiplatform (`org.jetbrains.compose.*`), jamais `androidx.compose.*` ni imports platform-specific
- **Offline-first** : SQLDelight local + sync_queue pour journal/poids/hydratation
- **Langue** : Code en anglais, UI/commentaires en francais
- **Tests** : Kotest (shared), Ktor test engine + Testcontainers (backend)
- **RTK** : Toujours prefixer les commandes Bash avec `rtk` pour reduire la consommation de tokens (ex: `rtk git status`, `rtk ./gradlew build`, `rtk docker ps`). Meme dans les chaines avec `&&`, chaque commande doit etre prefixee : `rtk git add . && rtk git commit -m "msg"`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hubinfourniret) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
