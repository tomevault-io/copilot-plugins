## monarch-store

> - `monarch-core` owns canonical package identity, metadata hydration, source ordering, discovery payloads, and detail payloads.

# MonARCH Store Cursor Rules

## Architectural invariants

### 1. Iron Core is the truth
- `monarch-core` owns canonical package identity, metadata hydration, source ordering, discovery payloads, and detail payloads.
- GTK must render backend models; it must not invent metadata, infer source truth, or recreate canonical merge logic.

### 2. GTK is the active frontend
- `src-tauri/monarch-gtk/` is the active product UI.
- Tauri/React in `src/` is legacy/reference-only unless explicitly marked otherwise.

### 3. Helper-only ALPM writes
- Only `monarch-helper` may write to `/var/lib/pacman` or run ALPM transactions.
- The frontend and `monarch-core` coordinate package operations, but the helper performs the write.

### 4. No partial-upgrade behavior
- Never run `pacman -Sy` alone.
- Repo installs and updates must preserve Arch-safe transaction rules.

### 5. AUR build split
- AUR builds remain unprivileged.
- Built artifacts are installed through the helper afterward.

### 6. One stable app, one canonical listing
- Stable repo/AUR/Flatpak variants of the same app merge into one canonical listing.
- Channel variants such as beta/canary/nightly/dev remain separate identities.

## UI rules

### Cards
- Outside search: compact Flathub-style cards
- Search only: screenshot-style cards
- Card content is limited to icon, title, short description, and ordered source badges

### Details
- Details is the action surface for install/open/remove/update state
- Source selection belongs in details, not on cards
- Details must reload source-specific payloads from Iron Core

### Icons and fallbacks
- Prefer the richest trusted app logo
- Avoid symbolic/monochrome placeholders when richer icons exist
- Fallbacks must look intentional and professional

## Documentation rules

- Public docs must describe current GTK truth
- Tauri docs or behaviors must be labeled legacy/reference
- Do not leave stale frontend claims in root docs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cpg716) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
