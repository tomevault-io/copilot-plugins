## manifest

> You are working on **Xtracta**, an open‑source XPath playground built with

You are working on **Xtracta**, an open‑source XPath playground built with
React 19 + TypeScript + Tailwind CSS + shadcn/ui on the frontend and
Node.js 20 LTS + Express on the backend.

General rules  
- Write all new code in **TypeScript ESM**.  
- Follow **Conventional Commits** for messages (feat:, fix:, refactor:, docs:)  
- Use **Prettier defaults** (printWidth = 100).  
- Favour functional programming & immutability where practical.  
- Include JSDoc/TSdoc for every exported function or component.  


# Terminal Command Standards


- **Absolute paths only** – Every file, directory, or executable referenced in a shell command **must** be written as an absolute path (starting with `/` on UNIX‑like systems). Avoid relying on the current working directory or implicit context.

  ```bash
  /usr/bin/node /home/user/xtracta/scripts/build.js
  ```

- **Single‑shot commands** – If a task requires multiple steps, chain them into **one** shell line using `&&` (fail‑fast) or `;` (run regardless), so the desired result is achieved in a single invocation.
  
  ```bash
  /usr/bin/env bash -c "mkdir -p /home/user/xtracta/dist && cp -r /home/user/xtracta/build/* /home/user/xtracta/dist/"
  ```

- **No implicit `cd`** – Do **not** use `cd` as a separate step. If directory context is essential, embed it:
  
  ```bash
  (cd /home/user/xtracta && npm run build)
  ```

- **Error handling** – Prefer `set -euo pipefail` at the start of scripts or rely on `&&` chaining so failures halt subsequent commands.

- **Platform consistency** – Use POSIX‑compliant syntax. Resolve environment variables (e.g., `$PROJECT_ROOT`) to an **absolute** path before execution.

These instructions apply to all AI‑generated terminal commands in Chat and Cmd‑K for this repository.

---
> Source: [mnhlt/Xtracta](https://github.com/mnhlt/Xtracta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
