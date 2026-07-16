## typescript-extension

> TypeScript conventions for TaskPlanner extension development


# TypeScript conventions

- This repo uses ESM/Node16-style imports: in `.ts` files, prefer relative imports that include the `.js` extension (e.g. `import { X } from './x.js'`).
- Keep `src/core/**` free of VS Code dependencies. Any `import * as vscode from 'vscode'` should live under `src/extension/**` (or test code that explicitly targets extension behavior).
- Tests are split by responsibility:
  - `src/test/core/**`: Vitest unit tests
  - `src/test/extension/**`: integration tests (`@vscode/test-cli`)
- Follow existing code style: single quotes, semicolons, 2-space indentation.

---
> Source: [smekai/taskplanner](https://github.com/smekai/taskplanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
