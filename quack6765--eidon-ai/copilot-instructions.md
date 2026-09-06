## eidon-ai

> - You may start the dev server (`npm run dev`) when needed.

# Project Instructions

### Dev Server

- You may start the dev server (`npm run dev`) when needed.
- The dev server uses a random port in the 3000-4000 range to support multiple worktrees.
- **Before starting**, check if a `.dev-server` file exists in the project root.
  - If it exists, read the URL from it (first line) and use that for testing.
  - If the file exists but the server is not running (cannot connect), delete it and start fresh.
- After starting `npm run dev`, wait for the `.dev-server` file to appear, then read the URL from it.
- The `.dev-server` file format is:
  ```
  http://localhost:3127
  PID: 12345
  ```

---
> Source: [Quack6765/Eidon-AI](https://github.com/Quack6765/Eidon-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
