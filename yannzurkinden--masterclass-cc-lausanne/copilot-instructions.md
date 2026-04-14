## masterclass-cc-lausanne

> Application web de gestion de contacts/prospects (mini CRM) avec assistant IA intégré.

# LeadPulse — Mini CRM

## Description
Application web de gestion de contacts/prospects (mini CRM) avec assistant IA intégré.
Projet pédagogique pour une masterclass Claude Code.

## Architecture
- Frontend : React + Vite + TypeScript dans `/frontend`
- Backend : FastAPI (Python) dans `/backend`
- DB : SQLite (fichier `backend/leadpulse.db`)
- UI : shadcn/ui components

## Commandes
- Frontend : `cd frontend && npm run dev` (port 5173)
- Backend : `cd backend && uvicorn main:app --reload` (port 8000)
- Tout lancer : `make start` (depuis la racine)

## Conventions
- API REST, tous les endpoints sous `/api/`
- Auth par session cookie
- Réponses JSON avec structure standard FastAPI
- Composants React dans `/frontend/src/components/`
- Pages dans `/frontend/src/pages/`
- Modèles SQLAlchemy dans `/backend/models/`
- Routes FastAPI dans `/backend/routes/`

## Base de données
- SQLite via SQLAlchemy
- Migrations avec Alembic (config dans `backend/alembic/`)
- Les migrations sont appliquées automatiquement au démarrage du backend
- Après modification d'un modèle : `cd backend && alembic revision --autogenerate -m "description"` (ou `make migrate msg="description"`)
- Pour reset la DB : supprimer `backend/leadpulse.db` et relancer le backend
- `render_as_batch=True` est activé dans `alembic/env.py` (nécessaire pour SQLite)

## Style
- shadcn/ui pour tous les composants UI
- Tailwind CSS pour le styling custom
- Pas de CSS modules ni styled-components

## Compte démo
- Email : demo@leadpulse.ch
- Mot de passe : demo1234

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yannzurkinden) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
