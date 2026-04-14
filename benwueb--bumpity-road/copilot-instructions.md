## bumpity-road

> Checklist for adding a new feature end-to-end


# Adding a New Feature — Checklist

Follow these steps to add a new feature following existing patterns:

## 1. Prisma Model (`prisma/schema.prisma`)
- Add model with `@id @default(auto()) @map("_id") @db.ObjectId`, `createdAt`, `updatedAt`
- For token-based ownership: add `ownerToken String?`
- For auth-based ownership: add `userId String @db.ObjectId` and relation to User
- For images: `imageUrl String`, `imagePublicId String`
- Run `npx prisma generate` then `npx prisma db push` (stop dev server first on Windows)

## 2. Types (`src/types/feature.ts`)
- Define main type (dates as ISO strings, nested user as `{ id, name }`)
- Define `CreateInput` and `UpdateInput` types

## 3. API Route (`src/app/api/feature/route.ts`)
- GET: fetch all, include `isAdmin` in response if relevant
- POST: create entry; use `randomBytes(32).toString("hex")` for token-based ownership
- PATCH: update entry; verify ownership (token or userId)
- DELETE: verify ownership or admin; call `deleteCloudinaryImage()` for images
- Award badges: call `checkAndAward*Badges(userId)`, return `newBadges` array

## 4. Server Data Fetching (`src/lib/feature-server.ts`)
- Export `getFeatureData()` — fetch entries with Prisma, check session for `currentUserId`/`isAdmin`

## 5. Client Hook (`src/hooks/use-feature.ts`)
- Accept `{ initialEntries?, initialIsAdmin?, currentUserId? }`
- State: `entries`, `isAdmin`, `isLoading`
- CRUD functions with `useCallback`, optimistic deletes
- Emit badges on create via `emitBadgesEarned(data.newBadges)`

## 6. Components (`src/components/feature/`)
- `FeatureList.tsx` — main layout (grid + form + edit modal)
- `FeatureCard.tsx` — individual entry card with gradient overlay
- `FeatureForm.tsx` — create form with controlled inputs
- `FeatureEditModal.tsx` — edit modal wrapping `<Modal>` from `@/components/ui/Modal`
- `index.ts` — barrel exports

## 7. Page (`src/app/feature/page.tsx`)
- Use `PageHeader`, `Suspense` with skeleton, `mx-auto max-w-6xl p-4 md:p-6`
- Server component fetches data, passes to client `FeatureList`

## 8. Card Gradients
Import from `@/lib/ui-gradients`: `CARD_GRADIENTS.slate | .emerald | .sky | .violet | .rose | .amber`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BenWueb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BenWueb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
