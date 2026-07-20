## unla

> Unla consists of four services:

# Unla
Unla consists of four services:
1. `mcp-gateway`: The core data-plane gateway service. It routes client requests to MCP Servers or RESTful APIs. Think of it as the “Nginx” of the MCP domain.
2. `apiserver`: The control-plane service, also serves as the backend for the web UI. It handles configuration management.
3. `web`: A lightweight React-based admin frontend, used for managing services conveniently.
4. `mock-server`: A mock downstream service, used for simulating target systems. It’s mainly for testing and validating mcp-gateway behavior.

## Development Instructions
- The web frontend is built with ReactJS. Run it with: `npm run dev`
- The other three services are written in Go, and can be started with: `go run ./cmd/*/main.go`

## Development Guidelines
- All code comments must be in English, and only added when the logic is not self-explanatory. Avoid obvious or redundant comments.
- When starting a task:
- Refer to docs/.ai/SOP.server.zh-CN.md for backend/server guidance.
- Refer to docs/.ai/SOP.client.zh-CN.md for frontend/web guidance.
- For the release process, check docs/.ai/release.md.

## After Each Task – Checkpoints
Make sure to do all of the following after completing any task:
1. Stop background or async processes: Ensure that no service process is left running unintentionally. You may use port checks if needed to confirm cleanup.
2. For Go services:
    - If you made any modifications, recompile the service to ensure correctness.
    - Run the tests: `make test`
3. For Web frontend: If you made changes, lint the codebase: `npm run lint`

---
> Source: [AmoyLab/Unla](https://github.com/AmoyLab/Unla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
