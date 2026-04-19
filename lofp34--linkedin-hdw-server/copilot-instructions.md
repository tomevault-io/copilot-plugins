## linkedin-hdw-server

> - Le serveur MCP local (Express) expose `POST http://localhost:4000/prospect`

# Endpoint HTTP de prospection

- Le serveur MCP local (Express) expose `POST http://localhost:4000/prospect`
- Le frontend (`src/App.js`) appelle:
  - Production (Vercel): `/api/prospect`
  - Dév local: modifiez pour appeler `http://localhost:4000/prospect` si vous utilisez l’Express local

Exemple de payload:
```json
{ "nom": "openai" }
```
Réponse (extrait):
```json
{
  "nom": "...",
  "headline": "...",
  "location": "...",
  "url": "...",
  "image": "...",
  "urn": "fsd_profile:...",
  "experience": [ ... ],
  "education": [ ... ],
  "skills": [ ... ]
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lofp34) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
