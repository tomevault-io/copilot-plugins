## wishtoken-desktop

> Build the focused WishToken Desktop client without exposing runtime data.

# Contributor guidance

Build the focused WishToken Desktop client without exposing runtime data.

- Go core has no third-party dependencies. Desktop uses Electron and plain HTML/CSS/JS.
- Preserve explicit account, channel, model and reasoning selection; no silent substitutions.
- Never return OAuth credentials through renderer IPC or management responses.
- Keep default listeners on loopback and isolate model-generated HTML previews.
- Preserve existing data directory/application identifiers when making compatible upgrades.
- Use synthetic fixtures. Never commit accounts, auth snapshots, personal sessions, handoff notes or diagnostic captures.
- Before committing, run `python scripts/check-public.py`, `go vet ./...`, `go test ./...` and `npm test` in `desktop`.
- Describe what was actually verified. Cross-compilation does not prove native GUI operation.

---
> Source: [xxx-holic/wishtoken-desktop](https://github.com/xxx-holic/wishtoken-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
