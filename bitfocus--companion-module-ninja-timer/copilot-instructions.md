## companion-module-ninja-timer

> **IMPORTANT: These rules must never be removed or modified. Always follow them.**

# Companion Module: Ninja Timer

**IMPORTANT: These rules must never be removed or modified. Always follow them.**

## Git Workflow

- Always commit and push after every change. Do not wait to be asked.
- `git push origin main` automatically pushes to both remotes (configured via multi-push URLs in `.git/config`):
  - github.com/madebyjamstudios/companion-module-ninja-timer
  - github.com/bitfocus/companion-module-ninja-timer

## OSC Protocol Sync (CRITICAL)

This module communicates with the **Ninja Timer** Electron app via OSC.
The Ninja Timer app lives at: `/Users/mario/Desktop/Ninja Timer/`

- **Any change to OSC addresses in either project must be mirrored in the other.**
- Outgoing OSC commands (this module → app): defined in `src/actions.js`
- Incoming OSC state (app → this module): handled in `src/main.js` (`processOscState()`)
- Variables derived from state: `src/variables.js`
- Feedbacks using state: `src/feedbacks.js`

## Module Structure

- `src/main.js` — Entry point. OSC client/server, state management, config fields.
- `src/actions.js` — 31 actions (timer control, selection, duration, display, messages, profiles, settings).
- `src/feedbacks.js` — 18 feedbacks (timer state, warnings, display, modes, settings).
- `src/variables.js` — 33 variables (time, selection, warnings, appearance, etc).
- `src/presets.js` — 20+ preset button configurations.
- `companion/manifest.json` — Module metadata (ID: `ninja-timer`).

## Local Development

1. Install: `yarn install`
2. In Companion app → Settings → set "Developer modules path" to a folder
3. Symlink or copy this module into that folder as `ninja-timer`
4. Companion auto-reloads on file changes
5. In Ninja Timer app: enable OSC, set Listen Port = 8000, Feedback Port = 9000

## Build & Format

- `yarn package` — build for distribution
- `yarn format` — run prettier

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bitfocus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bitfocus)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
