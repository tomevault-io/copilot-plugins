## secret-heart-pocket

> This is an intergenerational legacy platform called "Secret Heart Pocket" - a digital scrapbooking/storytelling app built with React, TypeScript, Firebase, and Vite.

# Secret Heart Pocket - Cursor Rules

## Project Context
This is an intergenerational legacy platform called "Secret Heart Pocket" - a digital scrapbooking/storytelling app built with React, TypeScript, Firebase, and Vite.

## Tech Stack
- Frontend: React 18 + TypeScript + Vite
- Backend: Firebase (Auth, Firestore, Storage, Functions)
- Styling: Tailwind CSS
- Deployment: Firebase Hosting with GitHub Actions

## Key Files
- Core App: `src/App.tsx`
- Components: `src/components/`
- Firebase Config: `src/lib/firebase.ts`
- Auth Bootstrap: `src/lib/authBootstrap.ts`
- Storage Utils: `src/lib/storage.ts`

## Development Guidelines
1. Maintain TypeScript strict mode
2. Use Firebase best practices for security rules
3. Follow existing component patterns
4. Test auth flows thoroughly
5. Optimize for mobile/responsive design

## When Working on This Project
- Reference `PROJECT_STATE.md` for current status
- Check `docs/archive/` for troubleshooting guides if needed
- Security rules in `firestore.rules` and `storage.rules`
- CI/CD via `.github/workflows/firebase-deploy.yml`

## Cost-Saving Tips
- Only load files you need to modify
- Use @mentions for specific files
- Batch multiple questions together
- Trust git status unless errors occur

---
*For the full Origami Protocol methodology, see `ORIGAMI_PROTOCOL.md` and `ORIGAMI_PROTOCOL_WHITEPAPER.md`*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cyserman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
