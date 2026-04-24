## idea8

> - `POST /auth/register` — Register new user

# API Routes

## Auth
- `POST /auth/register` — Register new user
- `POST /auth/login` — Login, returns JWT
- `GET /auth/me` — Get current user, tier, account type
- `GET /auth/profile` — Get user profile
- `POST /auth/google` — Google OAuth

## Ideas
- `POST /ideas/generate` — Generate new ideas (tier-aware)
- `GET /ideas/all` — List all user ideas
- `POST /ideas/validate` — Validate/refine idea
- `POST /ideas/{id}/deepdive` — Deep dive analysis
- `GET /ideas/{id}` — Get idea by ID
- `PUT /ideas/{id}` — Update idea
- `GET /ideas/{id}/deepdive_versions` — List versions

## Teams/Collab
- `POST /collaboration/ideas/{id}/collaborators` — Add collaborator
- `POST /collaboration/ideas/{id}/change-proposal` — Propose change

## Onboarding
- `POST /profile/onboarding` — Complete onboarding

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rkendel1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
