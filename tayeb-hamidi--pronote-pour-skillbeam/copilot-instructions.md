## pronote-pour-skillbeam

> - **Monorepo** : Next.js frontend (`/apps/web`), FastAPI microservices (`/services/*`), code Python partagé (`/shared/python/`)

# Notes projet — pronote-pour-skillbeam-lite

## Architecture
- **Monorepo** : Next.js frontend (`/apps/web`), FastAPI microservices (`/services/*`), code Python partagé (`/shared/python/`)
- **Docker Compose** : tous les services tournent en conteneurs sur `http://localhost:3784`

## Règle critique : TOUJOURS rebuild Docker après modification backend
Après toute modification dans `/shared/python/` ou `/services/`, il faut reconstruire les conteneurs Docker :
```bash
docker compose up -d --build --no-deps generate worker export api-gateway
```
Sans rebuild, les conteneurs continuent à exécuter l'ancien code.

## Fichiers clés
- `shared/python/shared/generation/templates.py` — pipeline de génération LLM (prompts, normalisation, paires matching)
- `shared/python/shared/exporters/pronote_xml_exporter.py` — export XML Pronote (a sa propre copie des fonctions de normalisation)
- `apps/web/app/page.tsx` — page principale frontend
- `apps/web/app/globals.css` — styles CSS custom
- `apps/web/components/tile.tsx` — composant tuile de sélection
- `apps/web/components/conversion-overlay.tsx` — overlay de progression

## Points d'attention
- `templates.py` et `pronote_xml_exporter.py` ont des fonctions dupliquées (`_normalize_matching_left_candidate`, `_coerce_matching_definition`, etc.) — toute correction doit être appliquée aux DEUX fichiers
- Les regex de split French (`que\b`, `qui\b`, `dont\b`) doivent avoir des word boundaries des deux côtés (`\bque\b`) pour ne pas couper les mots comme "analogique", "numérique"
- Le prompt LLM interdit les définitions commençant par "est"/"sont"
- La déduplication des paires matching est globale entre questions (via `used_pair_keys`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tayeb-hamidi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Tayeb-hamidi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
