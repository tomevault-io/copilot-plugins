## axon-face

> - Chat model embeds into parent Bubble Tea models — it is a component, not a standalone app

@AGENTS.md

## Conventions
- Chat model embeds into parent Bubble Tea models — it is a component, not a standalone app
- Styles are configurable via lipgloss; do not hardcode colors or dimensions
- Streaming updates arrive as Bubble Tea messages — use the standard `Update`/`View` cycle
- Entry types represent chat messages; session management tracks conversation state

## Constraints
- Depends on axon-loop, axon-talk, axon-tool only — no HTTP, no server dependencies
- This is a TUI library — do not add web handlers, REST endpoints, or server code
- Do not import axon (the server toolkit) — face is purely terminal-side
- Consumer apps (imago, vita) compose this; do not add app-specific logic here

## Testing
- `go test ./...` — all tests must pass
- `go vet ./...` — must be clean
- Test models by sending Bubble Tea messages and asserting on state and `View()` output

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/benaskins) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
