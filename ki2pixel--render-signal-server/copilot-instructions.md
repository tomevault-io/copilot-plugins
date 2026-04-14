## render-signal-server

> Automatic skill detection and routing matrix for workflow_mediapipe based on pattern matching and priority hierarchy


# Patterns de Détection pour Skills Integration

Cette matrice définit les patterns de détection automatique pour l'invocation des skills spécialisés.

| Pattern Détecté (FR/EN) | Skill / MCP Cible | Priorité |
| :--- | :--- | :--- |
| `vérifier config`, `état store`, `redis file`, `config sanity`, `check config` | **check-config** | 1 |
| `debug`, `bug`, `crash`, `error`, `troubleshoot`, `systematic debugging` | **debugging-strategies** | 1 |
| `memory bank`, `docs sync`, `audit docs`, `analyse memory` | **docs-sync-automaton** | 3 |
| `docs`, `readme`, `writing`, `technical writing`, `markdown` | **documentation** | 2 |
| `gros fichier`, `massive file`, `chirurgical`, `edit block` | **fast-filesystem** | 2 |
| `json`, `path`, `structure`, `inspect`, `valeur`, `clé` | **json-query** | 2 |
| `magic link`, `auth`, `token`, `revocation`, `security` | **magic-link-auth-companion** | 3 |
| `r2 transfer`, `cloudflare r2`, `file offload`, `transfer pipeline` | **r2-transfer-service-playbook** | 3 |
| `redis config`, `persistence`, `store sync`, `config drift`, `audit redis` | **redis-config-guardian** | 1 |
| `routing rules`, `rules engine`, `webhook rules`, `dynamic routing` | **routing-rules-orchestrator** | 3 |
| `test`, `pytest`, `suite`, `coverage`, `unit test` | **run-tests** | 2 |
| `js module`, `frontend module`, `es6 module`, `scaffold js` | **scaffold-js-module** | 3 |
| `python service`, `singleton service`, `backend service`, `scaffold python` | **scaffold-service** | 3 |
| `réflexion`, `think`, `logique`, `architecture`, `analyser` | **sequentialthinking** | 1 |
| `tâche`, `task`, `backlog`, `planification`, `roadmap` | **shrimp-task-manager** | 1 |
| `test matrix`, `pytest suites`, `unit redis r2`, `resilience test` | **testing-matrix-navigator** | 2 |
| `dashboard`, `ux`, `frontend`, `webhook panel`, `autosave` | **webhook-dashboard-ux-maintainer** | 3 |
| `render`, `déploiement`, `render.com`, `service creation`, `monitoring`, `orchestration` | **render-deployment-manager** | 3 |
| `git commit`, `push`, `commit push`, `git automation`, `stage changes` | **commit-push** | 2 |

## Auto-Loading Logic

When patterns detected, automatically load:
```
fast_read_file(".windsurf/skills/[SKILL_NAME]/SKILL.md")
```

## Multi-Skill Support

For complex requests, combine multiple skills based on pattern detection priority.

## Notes d'Implementation
- **Priorité 1** : Invocation automatique obligatoire
- **Priorité 2** : Invocation recommandée selon contexte
- Patterns sensibles à la casse et aux accents français

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ki2pixel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
