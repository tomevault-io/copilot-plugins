## constrictor

> > This file is auto-read by Claude Code at the start of every session. Update it after every coding session.

# CLAUDE.md — Constrictor

> This file is auto-read by Claude Code at the start of every session. Update it after every coding session.

---

## Project Summary

Constrictor is a local-first, encrypted password manager PWA. It stores passwords and secure notes in IndexedDB, encrypted with AES-256-GCM. Authentication requires an optional biometric check (Face ID / Touch ID) + 6-digit PIN + master password. No cloud, no recovery, no backdoors.

## Tech Stack

- React 18 + TypeScript + Vite (PWA)
- Dexie.js (IndexedDB)
- Web Crypto API (AES-GCM + PBKDF2)
- React Router
- CSS with Liquid Glass-inspired styling (backdrop-filter, glassmorphism)

## Key Architecture Decisions

- **Encryption key derivation:** PIN + master password → PBKDF2 → AES-256-GCM key. Salt stored in `meta` table. Key held in memory only while unlocked.
- **Verification:** A known string is encrypted during setup and stored. On login, decrypt it to verify credentials — never store credentials themselves.
- **Individual field encryption:** Each field (siteName, username, password, title, content) is encrypted separately before storage.
- **Lock behavior:** Configurable — lock on app close (default) or lock on background. Locking wipes the key from memory.
- **No recovery:** If the user forgets credentials, data is unrecoverable. This is intentional.
- **Note categories:** User-created categories with assigned colors from a fixed palette. Each color maps to a tinted glass rgba value. Category color is stored unencrypted; category name is encrypted. Notes have an optional categoryId foreign key.
- **Theme system:** CSS custom properties (variables) defined in `index.css`, toggled via `data-theme` attribute on `<html>`. Three-way toggle (System/Light/Dark) stored in `meta` table as `appearance`. System mode uses `prefers-color-scheme` media query + `matchMedia` listener. Theme context provided by `useTheme` hook.
- **Biometric authentication (Phase 9):** Optional Face ID / Touch ID via WebAuthn API as the first unlock gate before PIN + password. Uses `navigator.credentials.create()` for enrollment and `navigator.credentials.get()` for verification with `authenticatorAttachment: 'platform'` and `userVerification: 'required'`. Credential ID stored in `meta` table (not sensitive). WebAuthn challenge is random and not server-verified — used purely for local biometric gating. Falls back gracefully when device lacks biometric hardware or page is served over HTTP.

## Design System

- Dark/Light mode with System preference support, Liquid Glass-inspired UI
- Ambient color orbs (teal, indigo, rose radial gradients) behind content give backdrop-filter something to blur
- All glass surfaces are white/neutral — no colored tinting. Colors only on: password avatars (gradients), note category chips, error states
- Glass effects (backdrop-filter, translucent backgrounds, inset highlights) on navigation/control elements AND password list items (glass cards)
- Content areas clean and readable
- Background: dark gradient (navy/charcoal) in dark mode, light gray in light mode
- Min touch targets: 44x44px

## File Structure

```
src/
├── components/      UI components (PinPad, GlassCard, SearchBar, etc.)
├── pages/           Route-level pages (SetupFlow, LockScreen, PasswordsPage, etc.)
├── services/
│   ├── crypto.ts      PBKDF2 key derivation, AES-GCM encrypt/decrypt
│   ├── db.ts          Dexie database schema and operations
│   ├── auth.ts        Login, lock, key lifecycle management
│   └── biometrics.ts  WebAuthn Face ID / Touch ID enrollment + verification
├── hooks/
│   ├── useAuth.ts   Auth state context, visibility/close lock listeners
│   └── useTheme.tsx Theme context (System/Light/Dark), data-theme on <html>
├── styles/
│   └── glass.css    Liquid Glass CSS utility classes
├── App.tsx          Router + auth gate (setup vs lock vs main)
└── main.tsx         Entry point
```

## Dexie Schema

```typescript
meta:           key        // stores salt, verificationToken, lockBehavior, setupComplete, appearance
passwords:      ++id, siteName, username, password, dateAdded, dateModified
noteCategories: ++id, name, color, order, dateAdded
notes:          ++id, categoryId, title, content, dateAdded, dateModified
```

Meta keys include: `salt`, `verificationToken`, `lockBehavior`, `setupComplete`, `appearance`, `noteSortMode`, `biometricsEnabled`, `biometricCredentialId`.

Note: siteName/username/password/title/content/category name are all encrypted strings at rest. Category `color` is NOT encrypted (no sensitive data, needed for rendering).

## Development Phases

| Phase | Status | Description |
|-------|--------|-------------|
| 1 | NOT STARTED | Project scaffold, Dexie schema, setup flow, lock screen, key derivation |
| 2 | NOT STARTED | Encryption utilities, per-field encrypt/decrypt, key-in-memory management |
| 3 | NOT STARTED | Passwords CRUD, search, alphabetical list, copy, show/hide |
| 4 | NOT STARTED | Secure notes CRUD, date-sorted list |
| 5 | COMPLETE | Settings: lock behavior, change PIN, change master password |
| 6 | COMPLETE | Encrypted export/import (.constrictor files), brute-force lockout on lock screen |
| 7 | COMPLETE | PWA manifest, icons, UI polish, iPhone + Mac testing |
| 8 | COMPLETE | Drag-and-drop notes into categories, note sort modes (Date/Manual), category ordering follows sort mode |
| 9 | COMPLETE | Biometric authentication (Face ID / Touch ID) as optional first unlock step via WebAuthn |

## (Phase 8 — Just Before Launch)

- **Drag-and-drop notes into categories:** Drag a note onto a category chip to reassign it. Works in both sort modes.
- **Note sort modes:** New setting — Date (default, sort by dateModified) or Manual (user-defined order via drag). Adds `order` field to notes table and `noteSortMode` to meta table.
- **Category ordering follows sort mode:** Date mode → categories sort by dateAdded. Manual mode → categories reorderable via drag (uses existing `order` field).

## Current Session State

**Last session:** Phase 9 — Biometric authentication (Face ID / Touch ID) via WebAuthn as optional first unlock step. New biometrics.ts service, lock screen biometric stage, Settings toggle. Three-stage unlock: biometric → PIN → password.
**Next step:** All phases complete. Testing, bug fixes, final polish.

## Commands

```bash
npm run dev          # Start dev server
npm run build        # Production build
npm run preview      # Preview production build
```

## Rules

1. Always read `constrictor-project-spec.md` for full feature details before implementing.
2. Never store raw PIN or master password anywhere — only the derived key, and only in memory.
3. All password/note fields must be encrypted before writing to Dexie.
4. Encryption key must be wiped from memory on lock.
5. Use Web Crypto API — no third-party crypto libraries.
6. Glass styling goes on navigation/controls only, never on content lists.
7. Update this file's "Development Phases" table and "Current Session State" after every session.
8. Explain code logic and decisions to the user — learning is a priority.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/revant321) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
