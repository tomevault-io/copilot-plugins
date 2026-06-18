## document-dashboard

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# DocDash Agent Surface

For DocDash registry or document-management work, load `.agents/skills/docdash/SKILL.md`.

- Prefer the project-scoped `docdash` MCP server from `.mcp.json`.
- Fall back to `npm run docdash ...` from the repo root when MCP is unavailable.
- For global Codex availability outside this repo, run `npm run codex:install` once from the cloned DocDash repository.
- Do not edit `.docdash/registry.json` by hand; use DocDash interfaces instead.

---
> Source: [5eanxlee/Document-Dashboard](https://github.com/5eanxlee/Document-Dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
