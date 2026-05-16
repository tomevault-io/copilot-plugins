## polybuild-v3

> > Conventions de développement de POLYBUILD lui-même.

# AGENTS.md — polybuild-core

> Conventions de développement de POLYBUILD lui-même.
> Pour les projets utilisateurs qui consomment POLYBUILD, voir leur propre `AGENTS.md`.

## 0. Scope

`polybuild-core` est l'orchestrateur multi-LLM. Python 3.11+, asyncio strict, uv, pas de framework lourd.

## 1. Non-negotiable Constraints

- Pas de LangChain, pas de LlamaIndex, pas de framework agentic externe
- Pas de Node.js, pas de Go, pas de Rust en prod
- ruff + mypy --strict + pytest + pre-commit + bandit + gitleaks
- SQLite WAL en dev, immutable en prod (vector_store)
- Tous les appels réseau sous timeout (httpx + asyncio.wait_for)
- Aucune donnée nominative santé en prompt envoyé hors infrastructure locale ou EU certifiée

## 2. Architecture Invariants

- Adapter pattern unique (BuilderProtocol ABC) pour tous les modèles
- Output normalisé Pydantic v2 (BuilderResult, GateResults, AuditReport, etc.)
- Aucune voix Phase 2 ne sait quelles autres voix tournent (no cross-talk)
- Critic ≠ Fixer ≠ Verifier pour findings P0 (familles strictement différentes)
- Spec hashée SHA-256 après Phase 0c, vérifiée Phase 6 (anti-drift)
- Checkpoint atomique (tmp + rename) à chaque phase, jamais de commit partiel
- Repo dédié : aucune dépendance d'un projet utilisateur ne fuit ici

## 3. Coding Conventions

- Pydantic v2 typed contracts pour TOUTE structure de données passant entre phases
- Pas d'`except` bare, jamais
- Pas de `print()` — utiliser `structlog`
- snake_case Python, UPPER_SNAKE constantes, PascalCase classes
- Type hints partout (mypy --strict doit passer)
- async def pour toute I/O (subprocess, HTTP, fichiers > 10 KB)
- Imports relatifs interdits, toujours `from polybuild.x import y`
- Docstrings Google style sur toutes les fonctions publiques

## 4. Test Requirements

- Couverture pytest ≥ 80% sur `src/polybuild/`
- Scenarios obligatoires : happy / edge / failure pour chaque phase
- Smoke tests CLI hebdomadaires (`polybuild test-cli`)
- Gold prompts regression : 5 profils représentatifs minimum
- Property tests (hypothesis) pour `phase_3b_grounding` (parsing AST robuste)
- Mocks limités : ratio mock/real ≤ 40%, sinon refactor

## 5. Security & Privacy

- Aucun secret en code (gitleaks pre-commit, `.gitleaks.toml` configuré)
- API keys dans `~/.polybuild/secrets.env` chmod 600 (Round 4 à finaliser)
- bandit -ll en CI, severity HIGH = build break
- Privacy Gate (Phase -1) bloquante pour profils medical_*
- Routing OpenRouter interdit pour `risk_profile.excludes_openrouter == True`
- Mistral EU = endpoint api.mistral.ai DIRECT (pas OR — souveraineté)

## 6. Known Failure Patterns

| Pattern | Example | Prevention | Source |
|---|---|---|---|
| Convergent Hallucination | Multiple voices invent same fake API | Grounding AST Phase 3b + ≥2 = disqualification | Round 1 |
| Self-Fix Bias | Same family critiques and fixes | Critic≠Fixer≠Verifier families | Round 3 |
| Auditor Laziness | Audit returns 0 findings in <60s | Reject + retry with another auditor | Round 2 |
| Spec Drift Mid-run | Spec mutated during run | SHA-256 hash verified Phase 6 | Round 1 |
| CLI Forfait Throttle | Saturating concurrent calls | concurrency_limiter (Round 4) | Round 3 |
| OR for Sensitive Data | Health data through US infra | excludes_openrouter flag + filter | Round 2 |
| DeepSeek V3.2 INT4 NAS | 685B → 340GB impossible on 18GB NAS | Eliminated from local options | Round 3 |

## 7. Active ADRs

| ADR | Rule | Status |
|---|---|---|
| 0001 | Repo dédié `polybuild-core` (pas monorepo) | Accepted |
| 0002 | DeepSeek V3.2 INT4 écarté comme option locale | Accepted |
| 0003 | Mistral EU via api.mistral.ai direct, pas OpenRouter | Accepted |
| 0004 | DeepSeek V4-Pro intégré comme 6ème modèle de l'équipe consultative | Accepted |

## 8. Expiring Rules (TTL)

| Rule | Added | Expires | Owner |
|---|---|---|---|
| 5 placeholders Round 4 (Phase -1, gates domain, concurrency, Phase 8, secrets) | 2026-05-03 | 2026-06-03 | reddie |
| Verifier strict prompt à valider sur 5 runs réels avant fixation | 2026-05-03 | après 5 runs | reddie |

## 9. Modèles consultatifs (équipe round 4 — 6 modèles)

Pour les décisions architecturales majeures, consulter ces 6 modèles via prompts dans markdown code block :

1. `claude-opus-4.7` (CLI Claude Code) — architecte, contexte massif
2. `gpt-5.5` (Codex CLI) — exécution agentique, terminal-bench champion
3. `gemini-3.1-pro` (Gemini CLI) — vue holistique repo, ctx 2M
4. `kimi-k2.6` (Kimi CLI) — créatif, idioms alternatifs, swarm
5. `deepseek/deepseek-v4-pro` (OR) — raisonnement transparent, algo strict
6. `x-ai/grok-4.20` (OR) — adhérence stricte, low hallucination

DeepSeek a rejoint l'équipe au round 4 (avant : 5 modèles). Conséquence : seuil consensus passe de 4/5 à 5/6 pour les changements bloquants.

---
> Source: [reddepot/polybuild_v3](https://github.com/reddepot/polybuild_v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
