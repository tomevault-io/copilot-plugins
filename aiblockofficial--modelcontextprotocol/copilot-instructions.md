## plans-for-context

> The `/plans` directory contains a `masterplan.md` and step plans for each task.

The `/plans` directory contains a `masterplan.md` and step plans for each task.
Always reference these plans for project context.

Plan-writing guidance (concise):
- Guidance-first: describe steps and intent; avoid full file contents or long code blocks.
- Minimal commands: include only essential commands (e.g., `uv` installs, run/test) for clarity.
- Structure: use Phase sections (e.g., Tooling & Architecture; Abstract Execution Protocol) with Purpose and Action bullets.
- References: cite official docs/links relevant to the step (MCP, transports, SDKs).
 - References: cite official docs/links relevant to the step (MCP, transports, SDKs, Railway railpack).
- Dependencies: list package names; avoid pinning unless necessary; dev extras grouped per step.
- Environment: follow 12‑factor; provide `.env.example` guidance (no secrets in plans).
 - Deployment: include minimal guidance for `railpack.json` (install with `uv sync`, start with `uv run uvicorn ... --port $PORT`).
- Security & errors: note privileged ops and error conventions at spec level; implement later.

---
> Source: [AIBlockOfficial/modelcontextprotocol](https://github.com/AIBlockOfficial/modelcontextprotocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
