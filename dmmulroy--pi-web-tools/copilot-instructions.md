## pi-web-tools

> - Parse external data at boundaries before passing it inward.

# Pi Web Tools

## Guardrails

- Parse external data at boundaries before passing it inward.
- Preserve SSRF protections, URL credential redaction, response limits, and output truncation.
- Model expected failures with the local `Result` type; throw at the Pi adapter only to mark a tool execution as failed.
- Put runtime libraries in `dependencies` and Pi core packages in `peerDependencies`.
- Run `npm run check` after changes.

---
> Source: [dmmulroy/pi-web-tools](https://github.com/dmmulroy/pi-web-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
