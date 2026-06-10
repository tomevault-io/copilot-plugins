## stamply

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Stamply** — A mobile-first web app for stamp collectors. Users can photograph and catalog their stamps, browse a community feed, and trade stamps with other collectors.

## Tech Stack

- **Framework**: Next.js (App Router, latest)
- **Styling**: Tailwind CSS (custom color scheme — Indigo `#6366F1` primary, `#FAFAFA` background)
- **Backend**: Firebase — Auth + Firestore only (Storage is NOT used)
- **Images**: Compressed to base64 JPEG via `compressImage()` in `src/lib/utils.ts`, stored directly in Firestore (max 600px, 75% quality ≈ 50–100KB per image)
- **Deploy**: Vercel

## Commands

```bash
npm run dev      # Start dev server (http://localhost:3000)
npm run build    # Production build + TypeScript check
npm run lint     # ESLint
```

## Project Structure

```
src/
  app/
    (auth)/login/             # Login + Register (Google + Email/Password)
    (main)/layout.tsx         # Auth guard — redirects to /login if unauthenticated
    (main)/page.tsx           # Home (stub)
    (main)/inventory/
      page.tsx                # My stamps grid + add/edit/delete
      [uid]/page.tsx          # Another user's public inventory + Trade button
    (main)/community/
      page.tsx                # Posts feed with pagination
      [postId]/page.tsx       # Post detail + comments
    (main)/trade/page.tsx     # 3 tabs: Listings / Incoming / Outgoing requests
    (main)/profile/
      page.tsx                # Own profile + sign out
      [uid]/page.tsx          # Public profile: avatar, bio, stamp preview, View Inventory + Trade buttons
  components/
    BottomNav.tsx             # Fixed 5-tab bottom navigation
    stamps/StampCard.tsx      # Stamp card (owner menu / trade button)
    stamps/StampForm.tsx      # Bottom-sheet add/edit form
    community/PostCard.tsx    # Post card with like/comment, links to /profile/[uid]
    community/CreatePost.tsx  # Bottom-sheet create post (attach stamps from inventory)
    trade/TradeOfferModal.tsx # Bottom-sheet send trade offer (pick stamp + message)
  context/AuthContext.tsx     # Global auth state (firebaseUser + Firestore user)
  hooks/useAuth.ts            # onAuthStateChanged + auto-create/fix Firestore user doc
  lib/
    firebase.ts               # Firebase init (client-safe lazy getters)
    utils.ts                  # cn() + compressImage()
    firestore/
      users.ts                # getUser, createUser, updateUser
      stamps.ts               # CRUD stamps (base64 imageURL, no Storage)
      posts.ts                # CRUD posts, likes, comments
      trades.ts               # trade_listings + trade_requests CRUD + acceptTradeRequest
  types/index.ts              # Shared TypeScript interfaces
```

## Firebase Notes

**No Firebase Storage** — all images stored as base64 data URLs in Firestore documents.  
Always call `compressImage(file)` from `lib/utils.ts` before writing any image to Firestore.

Firebase initializes lazily (client-side only) in `lib/firebase.ts` using `typeof window !== "undefined"` guards. All Firebase calls must be inside `"use client"` components or hooks.

### Auth — displayName Timing Issue
Email/Password signup: `onAuthStateChanged` fires before `updateProfile` completes.

**Fix applied in two places:**
1. `app/(auth)/login/page.tsx` — after `updateProfile`, explicitly calls `createUser()` with the correct name before navigating away. Does not rely on `onAuthStateChanged` to write the Firestore doc.
2. `hooks/useAuth.ts` — on every login, if Firestore `displayName` is null but `firebaseUser.displayName` is set, auto-repairs the document via `updateUser()`.

Never pass raw `user.displayName` to Firestore without a fallback — always use:
```ts
user.displayName || firebaseUser.displayName || ""
```

## Firestore Collections

```
users/{uid}           displayName, photoURL, email, bio, stampCount, createdAt
stamps/{id}           ownerId, ownerName, ownerPhotoURL, name, country, year,
                      condition, description, imageURL (base64), isPublic, createdAt
posts/{id}            authorId, authorName, authorPhotoURL, content, imageURLs[],
                      attachedStampIds[], likesCount, commentsCount, createdAt
  └─ comments/{id}    postId, authorId, authorName, authorPhotoURL, content, createdAt
  └─ likes/{uid}      uid (document existence = liked)
trade_listings/{id}   posterId, posterName, posterPhotoURL, offeredStampId,
                      wantedDescription, status (open|closed), createdAt
trade_requests/{id}   fromUserId, fromUserName, fromUserPhotoURL, toUserId,
                      offeredStampId, requestedStampId, listingId?, message?,
                      status (pending|accepted|declined), createdAt
```

## Firestore Composite Indexes Required

All 5 indexes must exist in Firebase Console → Firestore → Indexes (also defined in `firestore.indexes.json`):

| Collection | Fields | Used by |
|---|---|---|
| `stamps` | `ownerId` ASC + `createdAt` DESC | My inventory |
| `stamps` | `ownerId` ASC + `isPublic` ASC + `createdAt` DESC | Public user inventory |
| `trade_listings` | `status` ASC + `createdAt` DESC | Open listings feed |
| `trade_requests` | `fromUserId` ASC + `createdAt` DESC | Outgoing requests |
| `trade_requests` | `status` ASC + `toUserId` ASC + `createdAt` DESC | Incoming requests |

## Trade Flow

```
[Post listing]   User A posts listing (offeredStampId + wantedDescription)
[Make offer]     User B clicks "Make Offer" on listing → TradeOfferModal → picks own stamp → sends trade_request
[Direct trade]   User B visits /profile/[uid] or /inventory/[uid] → clicks "Trade" on a stamp → same TradeOfferModal

[Accept]         acceptTradeRequest() swaps ownerId on both stamps in Firestore + closes listing
[Decline]        declineTradeRequest() sets request status = "declined"
```

## Navigation Between Users

- **Community feed** → tap author name → `/profile/[uid]`
- **Trade listings** → tap poster name → `/profile/[uid]`
- **`/profile/[uid]`** → "View Inventory" button → `/inventory/[uid]`
- **`/inventory/[uid]`** → "Trade" button on any stamp → `TradeOfferModal`

## Design

Mobile-first, max-width 480px centered. Fixed bottom nav (5 tabs: Home / Inventory / Community / Trade / Profile). All modals are bottom-sheets sliding up from the bottom.

Color tokens: `bg-primary-500` (#6366F1 indigo), `bg-background` (#FAFAFA), `bg-muted` (#f4f4f5), `text-muted-foreground` (#71717a), `border-border` (#e4e4e7).

Bottom nav height is 72px — bottom-sheets and fixed bottom elements must add `pb-[88px]` or equivalent to avoid being hidden behind the nav.

---
> Source: [ymetdev/stamply](https://github.com/ymetdev/stamply) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
