## nodeblast

> Creator portfolio platform at nodeblast.dev. Shares Firebase project (dexnote-d7047) and user accounts with DexNote.

# NodeBlast

Creator portfolio platform at nodeblast.dev. Shares Firebase project (dexnote-d7047) and user accounts with DexNote.

## Tech Stack
- Vanilla JavaScript ES modules
- Firebase Auth (Google + GitHub)
- Firestore for data
- Firebase Storage for images
- Deployed on Vercel

## Architecture
- Single-page app, all JS in /js/, all CSS in /css/
- firebase-config.js auto-switches between dev (127.0.0.1) and prod by hostname
- Auth uses popup flow, authDomain stays as dexnote-d7047.firebaseapp.com (do NOT change this)

## Brand
- Name: NodeBlast (variable: keep easy to change)
- Tagline: "create. share. detonate."
- Primary color: #5AAA72
- Font: Outfit

## Key Rules
- Same user account system as DexNote — same users/ collection in Firestore
- New Firestore collections for NodeBlast: catalysts/, votes/
- Firebase Storage path for thumbnails: /catalysts/{userId}/{catalystId}/thumb

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dexdcimino) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
