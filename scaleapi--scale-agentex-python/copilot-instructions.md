## 05-permissions-and-tools

> Cursor agent permissions and allowed tools aligned with Claude settings


When invoking external tools or the terminal, follow these constraints:

- Web search is allowed when needed for docs and references
- Prefer fetching docs from `docs.temporal.io` when researching Temporal topics
- Allowed bash commands should go through Rye workflows:
  - `rye run pytest:*`
  - `rye run lint:*`
  - `rye run typecheck:*`
  - `rye run sync:*`
  - `rye run build:*`

Default to Rye; only use other tools when explicitly required by the codebase.

---
> Source: [scaleapi/scale-agentex-python](https://github.com/scaleapi/scale-agentex-python) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
