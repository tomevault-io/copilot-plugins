## ruggylab-os

> Guide rapide pour les agents qui interviennent sur RuggyLab OS.

# AGENTS.md

Guide rapide pour les agents qui interviennent sur RuggyLab OS.

## Projet

RuggyLab OS est un backend FastAPI pour workflows de laboratoire: patients, echantillons, resultats, reactifs, audit, rapports, DH36/HL7, imagerie, epidemiologie, pharmacie CMU et exports FHIR.

Architecture principale:

- `app/main.py`: creation FastAPI, middlewares, UI embarquee et health checks.
- `app/api/v1/api.py`: enregistrement des routers.
- `app/api/v1/endpoints/`: endpoints HTTP.
- `app/schemas/`: schemas Pydantic d'entree/sortie.
- `app/services/`: logique metier testable.
- `app/models/ruggylab_os.py`: modeles SQLAlchemy.
- `alembic/versions/`: migrations.
- `tests/`: tests unitaires, API, securite, migrations et metier.

## Commandes De Validation

Depuis la racine du depot:

```powershell
.\scripts\validate.ps1
```

Ou manuellement:

```powershell
python -m ruff check app tests
python -m bandit -q -r app -c pyproject.toml
python -m pip_audit -r requirements.txt --ignore-vuln PYSEC-2025-183
python -m pytest --tb=short -q
```

La suite complete prend environ 2 a 3 minutes en local et doit finir autour de `478 passed, 4 skipped`.

Pour une passe ciblee apres changement auth/API:

```powershell
.\scripts\validate.ps1 -Fast
```

Pour une passe visuelle du cockpit apres changement UI, demarrer l'app sur `127.0.0.1:8010`, puis lancer:

```powershell
node scripts\ui_check_playwright.js
```

Ce controle requiert Playwright cote Node et genere des captures ignorees dans `artifacts/ui-check/`.

## Demarrage Local

Windows:

```powershell
.\scripts\install.ps1
.\scripts\start.ps1
```

Manuel:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
copy .env.example .env
uvicorn app.main:app --reload
```

URLs utiles:

- Cockpit: `http://127.0.0.1:8000/app`
- Swagger: `http://127.0.0.1:8000/docs`
- Health: `http://127.0.0.1:8000/api/v1/health`

## Regles De Changement

- Preferer les patterns existants: endpoint fin, schema Pydantic, service metier, modele SQLAlchemy si besoin.
- Ajouter ou adapter des tests quand un comportement API, metier, securite ou migration change.
- Ne pas modifier `.env`, bases SQLite locales, backups, logs ou caches.
- Ne pas committer de poids ML dans `models/`; le modele paludisme peut etre absent en local et l'app utilise alors un stub heuristique.
- Garder les secrets hors du code. Utiliser `.env.example` comme documentation, pas `.env`.
- Pour les migrations, mettre a jour `app/models/ruggylab_os.py`, creer une migration Alembic, puis verifier avec `tests/test_alembic_migrations.py`.
- Pour les routes protegees, utiliser les dependances d'auth existantes dans `app/api/deps.py`.
- Pour les domaines FHIR/CMU/DH36, verifier les tests dedies avant et apres modification.

## Pieges Connus

- Les gros fichiers HTML dans `app/templates/` sont monolithiques; faire des edits tres localises et tester visuellement si l'UI change.
- Le listener DH36 ne doit pas demarrer en mode test.
- Le fallback IA paludisme n'est pas un vrai modele; ne pas le presenter comme inference clinique reelle.
- `Makefile` est surtout adapte aux environnements Unix. Sous Windows, privilegier les commandes PowerShell ci-dessus.
- `.claude/` contient des worktrees locaux d'agents et doit etre ignore lors de l'exploration du depot.

## Avant De Terminer

Verifier au minimum:

```powershell
git status --short
.\scripts\validate.ps1
```

Pour un changement de dependance ou de securite, ajouter:

```powershell
.\scripts\validate.ps1
```

---
> Source: [rommellnelson-ux/ruggylab-os](https://github.com/rommellnelson-ux/ruggylab-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
