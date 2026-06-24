## codestory

> CodeStory local grounding. Use repo evidence before source claims.


# CodeStory Grounding

Use CodeStory proactively for repository questions. Do not wait for the user to mention it by name.

Before making source claims, planning edits, choosing tests, or reviewing changes in this repository:

1. If the CodeStory MCP server is live, read `codestory://status` first.
2. Treat `server_version`, `server_executable`, `allowed_surfaces`, and `retrieval_mode` from status as runtime truth.
3. Use local graph surfaces only when their own `allowed_surfaces` entry allows them.
4. Use `packet`, `search`, or `context` only when that surface is allowed and `retrieval_mode=full`.
5. If MCP is missing, use `codestory-cli ready` or `doctor` as a repair/debug fallback.

---
> Source: [TheGreenCedar/CodeStory](https://github.com/TheGreenCedar/CodeStory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
