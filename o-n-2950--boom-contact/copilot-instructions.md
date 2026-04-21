## boom-contact

> > Lire CONTEXT.md + TODO.md avant toute action.

# boom.contact — Claude Instructions

> Lire CONTEXT.md + TODO.md avant toute action.

## Mission
boom.contact est la première app mondiale de constat amiable digital.
Deux conducteurs, un QR code, un PDF légal en 5 minutes. 42 langues. 50+ pays.

## Stack
- Frontend  : React 18 + Vite + TailwindCSS v4
- Backend   : Node.js + TypeScript + tRPC + Express
- Realtime  : Socket.io (session QR partagée)
- DB        : PostgreSQL (Railway)
- OCR       : Claude Vision API (Anthropic SDK)
- PDF       : pdf-lib
- Auth      : JWT (sessions éphémères par accident)
- Deploy    : Railway

## Architecture des sessions
Chaque accident crée une SESSION unique (nanoid).
- Conducteur A crée la session → reçoit un QR code
- Conducteur B scanne le QR → rejoint la session
- Les deux remplissent en parallèle (WebSocket sync)
- Les deux signent → PDF généré → envoyé aux deux

## Modules principaux
1. OCRService       — Claude Vision API → JSON structuré
2. SessionService   — Création/gestion sessions temps réel
3. FormCEA          — Formulaire standard CEA 17 champs
4. CarDiagram       — SVG interactif zones de choc
5. SketchCanvas     — Canvas libre croquis accident
6. SignatureService — Signature tactile dual
7. PDFGenerator     — Export PDF conforme CEA
8. I18nService      — 42 langues, 8 régions, RTL support

## Règles absolues
- Ne jamais casser une session en cours
- Toujours valider avec Zod avant traitement
- Données sensibles : chiffrement AES-256
- RGPD : sessions auto-détruites après 30 jours
- Build doit passer avant tout commit

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/O-N-2950) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
