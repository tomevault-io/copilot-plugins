## aviary

> - Aviary is an MCP, so you can connect to it via `go run ./cmd/aviary serve` and then connect on https://localhost:16677/mcp (read ~/.config/aviary/aviary.yaml for port, ~/.config/aviary/token has the bearer token).

# Developer Notes

- Aviary is an MCP, so you can connect to it via `go run ./cmd/aviary serve` and then connect on https://localhost:16677/mcp (read ~/.config/aviary/aviary.yaml for port, ~/.config/aviary/token has the bearer token).
- Always use the MCP to test features when developing
- Run `pnpm test:go` after any Go changes; run `pnpm lint` after any web changes; run `pnpm test` to run everything

---
> Source: [lsegal/aviary](https://github.com/lsegal/aviary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
