## rothex

> Use a free port for local web testing


# Testing Web Server Port

- When testing the web project locally, do not use the default/shared port `8000` unless the user explicitly asks for it.
- Prefer starting the test web server with port `0` so the OS assigns a free port.
- Prefer an unbuffered command so the assigned port is printed immediately: `python -u -m http.server 0 --directory .`
- After starting the server, read its startup output, capture the assigned port from the reported URL, and use that URL for browser-based testing.
- If the chosen server cannot bind to port `0`, choose a different free port instead of `8000`.
- Keep this behavior limited to testing workflows; do not change checked-in scripts or config unless the user explicitly asks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rshouker)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rshouker)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
