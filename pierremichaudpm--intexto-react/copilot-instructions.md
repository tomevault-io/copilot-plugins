## intexto-react

> Site d'information communautaire haïtien-québécois. Frontend React + Vite, backend Strapi (Railway), médias sur Cloudinary, déployé sur Netlify.

# CLAUDE.md — Intexto React

## Projet
Site d'information communautaire haïtien-québécois. Frontend React + Vite, backend Strapi (Railway), médias sur Cloudinary, déployé sur Netlify.

## Stack technique
- **Frontend** : React 18, Vite, Framer Motion, Plyr (audio/vidéo), react-i18next
- **CMS** : Strapi v5 (Railway) avec Cloudinary pour les médias
- **Déploiement** : Netlify (frontend), Railway (Strapi + PostgreSQL)
- **Proxy** : Express server.js proxie /api/* vers Strapi (contourne les pare-feux gouvernementaux)

## Architecture clé
```
src/
├── components/
│   ├── common/         # ContentModal, ContentCard, ResponsiveImage
│   ├── layout/         # Header, Footer, Hero
│   ├── sections/       # ContentGrid, LineupSection
│   └── seo/            # SEOHead, StructuredData
├── config/categories.js
├── context/ContentContext.jsx
├── services/
│   ├── cmsService.js   # Abstraction CMS (délègue à strapiService)
│   └── strapiService.js # Appels API Strapi, transformations
└── i18n/               # Traductions fr/en/ht
```

## Conventions
- Les vidéos utilisent **videoUrl** (URL YouTube ou externe). Le champ videoFile a été supprimé du schema Strapi.
- Les URLs YouTube sont détectées et rendues via iframe youtube-nocookie.com. Les autres URLs vidéo utilisent Plyr.
- Les lineups (manchettes) sont ordonnées côté Strapi FR, l'ordre est appliqué aux autres locales.
- Les images passent par Cloudinary (URLs absolues) ou Strapi local (URLs relatives préfixées).

## Commandes
```bash
npm run dev          # Dev local (Vite, port 5173)
npm run build        # Build production
node server.js       # Serveur Express avec proxy API (production)
```

## Repo GitHub
`git@github.com:pierremichaudpm/intexto-react.git` (branche main)

---
> Source: [pierremichaudpm/intexto-react](https://github.com/pierremichaudpm/intexto-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
