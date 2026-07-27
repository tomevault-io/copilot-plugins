## graysky

> A full-stack Bluesky client with React Native mobile app, Next.js web app, and push notification service.

# Graysky - Bluesky Client Monorepo

A full-stack Bluesky client with React Native mobile app, Next.js web app, and push notification service.

## Quick Commands

```bash
pnpm dev           # Start Expo mobile app
pnpm dev:both      # Start Expo + Next.js in parallel
pnpm dev:next      # Start Next.js web app only
pnpm dev:push      # Start push notification service
pnpm lint          # Run ESLint across all packages
pnpm format:fix    # Run Prettier formatting
pnpm typecheck     # TypeScript type checking
pnpm db:generate   # Generate Prisma client
pnpm db:push       # Push Prisma schema to database
```

## Project Structure

```
apps/
  expo/          # React Native mobile app (Expo 54, RN 0.81, React 19)
  nextjs/        # Next.js 16 web app (marketing/web presence)
  push-notifs/   # Backend push notification service

packages/
  api/           # tRPC router (shared API layer)
  db/            # Prisma database client and schema

tooling/
  eslint/        # Shared ESLint configs (flat config format)
  prettier/      # Shared Prettier config
  tailwind/      # Shared Tailwind config
  typescript/    # Base TypeScript config
```

## Key Technologies

- **Mobile**: Expo 54, React Native 0.81, Expo Router, NativeWind v2
- **Web**: Next.js 16, Tailwind CSS 4
- **API**: tRPC v11, Zod validation, SuperJSON serialization
- **Database**: Prisma ORM, MySQL (PlanetScale)
- **Build**: pnpm workspaces, Turbo (with remote caching)
- **Bluesky**: @atproto/api for protocol integration

## Package Dependencies

```
expo → @graysky/api, @graysky/tailwind-config
nextjs → @graysky/api, @graysky/db, @graysky/tailwind-config
push-notifs → @graysky/db
api → @graysky/db
```

## Database Models (Prisma)

- `User` - Base user by DID (Bluesky identifier)
- `TranslatablePost` / `PostTranslation` - Translation feature
- `Poll` / `PollVote` - Polls feature
- `PushToken` - Push notification device tokens
- `Mute` / `MuteList` - Mute functionality

## Environment Variables

Copy `.env.example` to `.env`. Key variables:
- `DATABASE_URL` - PlanetScale MySQL connection
- `GOOGLE_API_KEY` - Translation API
- `DEEPL_API_KEY` - Alternative translation
- Various RevenueCat, Sentry keys for mobile

## Code Conventions

- Strict TypeScript with `noUncheckedIndexedAccess`
- ESLint flat config with typescript-eslint and React Compiler
- Import order: react → next → expo → third-party → @graysky → relative
- tRPC for end-to-end type safety
- Lingui for i18n in mobile app (`pnpm extract` / `pnpm compile`)

## Build & Deploy

- **Mobile**: EAS Build for iOS/Android (`eas build`)
- **Web**: Vercel hosting
- **CI**: GitHub Actions runs lint, typecheck, build on PRs

---
> Source: [mozzius/graysky](https://github.com/mozzius/graysky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
