## chess-rebundled-co

> This project uses **pnpm** as the package manager. Always use pnpm commands instead of npm:

# Project Instructions for Claude

## Package Manager

This project uses **pnpm** as the package manager. Always use pnpm commands instead of npm:

```bash
# Install dependencies
pnpm install

# Add a package
pnpm add <package-name>
pnpm add -D <dev-package-name>  # dev dependency

# Run scripts
pnpm run build
pnpm run dev

# Workspace commands
pnpm -w frontend run build     # Run in frontend workspace
pnpm -w backend run build      # Run in backend workspace
```

## Project Structure

- `frontend/` - Next.js 15 app with React 19
- `backend/` - Fastify server with Socket.io

## Environment Variables

### Frontend (NEXT_PUBLIC_*)
- Firebase config: `NEXT_PUBLIC_FIREBASE_*`
- Analytics: `NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID`

### Backend
- `FIREBASE_*` - Firebase Admin SDK credentials
- `MONGODB_URI` - MongoDB connection string
- `GOOGLE_API_KEY` - Gemini API key

---
> Source: [chandreshkkhatri/chess-rebundled-co](https://github.com/chandreshkkhatri/chess-rebundled-co) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
