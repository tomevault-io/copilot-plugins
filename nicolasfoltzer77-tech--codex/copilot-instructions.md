## codex

> 1. Tous les tests passent (ne rien casser).

# AGENTS.md — Guide pour l’agent (Codex)

## Mission & Priorités
1. Tous les tests passent (ne rien casser).
2. Respecter l’archi et le style existants.
3. Petites PR (≈ < 400 lignes) avec docs et tests.
4. Zéro secret en clair, pas d’appels réseau non autorisés.

## Commandes standard
- Setup :
  ```bash
  python -m venv .venv && source .venv/bin/activate
  pip install -r requirements.txt
  pip install -r requirements-dev.txt
  ```
- Run : `uvicorn app.main:app --reload`
- Tests : `pytest -q`
- Lint/format : `ruff check . && ruff format .`

## Workflow attendu
1. Lire le code impacté et proposer un plan (3–5 puces).
2. Implémenter code + tests (happy path + erreurs).
3. Lancer lint + tests localement (aucun réseau réel, tout mocké).
4. Mettre à jour README/examples si besoin.
5. Ouvrir une PR (Conventional Commits) + checklist ci-dessous.

## Règles de code
- Python 3.11+, PEP8/PEP257, type hints.
- FastAPI + schémas Pydantic pour l’I/O. Logs via `logging`.
- Fonctions courtes, extraires helpers si nécessaire.

## Sécurité
- Variables d’environnement pour secrets (fournir `.env.sample` si besoin).
- Pas d’accès réseau dans les tests.
- Ajout de dépendances justifié dans la PR.

## Tests
- `tests/test_*.py`, déterministes (horloge figée si nécessaire).
- Couverture diff ≥ 90% si outil dispo.
- Inclure cas d’erreurs (422, 404).

## Git
- Branches : `feat/<slug>`, `fix/<slug>`, `chore/<slug>`.
- Commits : Conventional Commits.

## Modèle de PR
**Titre**: `feat: <résumé>`

**Contexte**
- Pourquoi : …
- Quoi : …

**Comment tester**
```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt -r requirements-dev.txt
pytest -q
```

**Checklist**
- [ ] Tests passent
- [ ] Lint/format ok
- [ ] README/docs à jour
- [ ] Pas de secrets / données perso

## Tâches types
- “Ajoute `GET /invoices` avec tests et docs.”
- “Refactor du module bot pour isoler la logique et augmenter la couverture.”

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nicolasfoltzer77-tech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/nicolasfoltzer77-tech)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
