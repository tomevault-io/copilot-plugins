## live-html

> Make any single-file HTML artifact editable in the browser with edits round-tripped to disk. Use when the user wants to revise a slide deck, dashboard, internal tool, or design-system viewer in place — clicking text to edit it, with Save writing back to the source file. Drop-in JS+CSS, no build, no server.


# live-html — edit HTML artifacts in place

A vanilla JS+CSS library that turns a static HTML file into an editable surface. Users click "edit on", revise text and theme tokens directly in the browser, hit Save, and the file on disk is overwritten via the File System Access API. No build step, no server, no framework.

## When to invoke

Use this skill when the user is about to ship — and then keep revising — a single-file HTML artifact they own:

- **Slide decks** authored in HTML (Reveal-free, single file, presenter is the editor).
- **Dashboards** that read from a static JSON blob and need copy/layout tweaks.
- **Internal tools and admin pages** where the audience is small and round-tripping edits beats a CMS.
- **Paper walkthroughs / explainers** where the author iterates on phrasing weekly.
- **Design-system viewers** — pages whose own CSS custom properties are the thing being edited.

The shape: one `.html` file the user opens locally, edits, saves. Same file. Days or weeks of iteration.

## When NOT to invoke

- **One-shot HTML** — a single email mockup, a throwaway prototype. Just edit the source.
- **Anything review-gated** — if changes need PR review, the canonical source is git, not the rendered file. Editing in-browser bypasses review.
- **Anything reading live data** — if the page mutates based on server state, in-place editing of the static markup is meaningless; you need a real CMS or app.
- **Multi-author concurrent edit** — see `recipes/upgrade-to-server.md` for the human+AI co-edit case; for human+human, reach for a real tool.

## Quick install

1. Copy `editor.css` into the page's `<head>`.
2. Paste `snippet.html` (the editor toolbar + history panel) immediately before `</body>`.
3. Drop `editor.js` after the snippet. Done — open the file, click "edit on", revise, Save.

Full walkthrough with gotchas: `recipes/add-to-existing.md`.

## How edits flow

Three states, in order of authority:

- **DOM is canonical.** While the page is open, the live DOM is the truth. Every edit mutates the DOM first; everything else is derived.
- **The op log is staging.** Each edit is appended to a `localStorage` op log (kinds: `content`, `theme`, `theme-switch`). The log survives reloads, supports undo/redo, and is what makes a half-finished session resumable. It is *not* the canonical record.
- **The disk file is committed.** Save serializes the current DOM (with active CSS-variable overrides baked into `<style id="theme-overrides">`), writes back to the original `.html` via the File System Access API, and clears the op log. The file is now the new ground truth.

This is git-shaped on purpose: working tree (DOM) → index (op log) → commit (file).

## Design system

The page exposes its visual identity as CSS custom properties on `:root` (or `html`). The library reads these, lets the user edit them via a generated theme picker, and writes deltas back into a `<style id="theme-overrides">` block on save. Optionally, the page declares named themes in a `<script type="application/json" id="themes">` block — switching themes is a single atomic op (`theme-switch`) that swaps the active overrides and updates `<html data-active-theme>`. See `ARCHITECTURE.md` for the four-layer model.

## When to escape this skill

- **Rich-text needs beyond `contenteditable`** — tables, media embeds, structured lists with constraints. Upgrade to Tiptap. See `TRADEOFFS.md`.
- **A real CMS** — multiple non-technical authors, permissions, drafts, scheduled publish. Out of scope.
- **Server-mediated merge** — when an AI agent and a human edit the same file concurrently, the local-only model breaks. See `recipes/upgrade-to-server.md`.

## Files in this skill

- `SKILL.md` — this file. Entry point and trigger conditions.
- `ARCHITECTURE.md` — the four-layer data model, op kinds, coalescing, persistence flow.
- `TRADEOFFS.md` — design decisions worth revisiting (Tiptap, single-file, snapshots, server, identity, namespacing).
- `editor.js` — the runtime: edit toggle, op log, undo/redo, theme picker, save.
- `editor.css` — toolbar, history panel, edit-mode outlines.
- `snippet.html` — the markup to paste before `</body>`.
- `design-system.html` — optional drop-in slide that exposes the page's CSS custom properties as an editable swatch grid.
- `recipes/add-to-existing.md` — bolt onto an existing HTML file in five minutes.
- `recipes/add-theme.md` — register a named theme.
- `recipes/upgrade-to-server.md` — when and how to move to a tiny local server.

---
> Source: [GindaChen/live-html](https://github.com/GindaChen/live-html) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
