## nero

> - Build a Personal Knowledge Management (PKM) and daily notes desktop application.

# AGENT BRIEFING

## Mission
- Build a Personal Knowledge Management (PKM) and daily notes desktop application.
- Organise entries by mirroring the user’s existing directory structure; the filesystem is the source of truth.
- Deliver a fast native-like experience by pairing a Go backend (Wails) with a React + Tailwind + daisyUI frontend.

## Current State
- Wails scaffold in place with the default Go entrypoints (`main.go`, `app.go`).
- Frontend bootstrapped with Vite/React and Tailwind; daisyUI (dark theme) is now wired in for component styling.
- Global styles keep Nunito as the primary font and rely on daisyUI tokens (`bg-base-200`, `btn`, etc.) for UI controls.

## High-Level Responsibilities
- **Backend (Go/Wails):** expose filesystem operations (browse, read/write markdown or text notes, manage daily note templates). Maintain cross-platform compatibility.
- **Frontend (React):** provide editors, directory navigation, and daily note dashboards. Use daisyUI components to ensure consistent dark-mode theming.
- **State & Data Flow:** treat the local directory tree as the authoritative datastore; cache only when necessary and sync back promptly to disk.
- **User Experience:** optimise for quick capture (daily note, scratch pad) and long-term organisation (tags, backlinks, search).

## Guardrails
- Preserve user data integrity: never delete or restructure directories without explicit confirmation.
- Work offline-first; avoid introducing network dependencies unless strictly required.
- Keep styling changes within Tailwind/daisyUI conventions to simplify theming updates.

## Next Milestones
1. Define the directory schema for PKM vs. daily notes and surface it via Go bindings.
2. Implement a navigation sidebar and daily note creation flow in the React UI.
3. Add automated tests for filesystem operations and UI state transitions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/papered)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/papered)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
