## kaba-frontend

> Cuando se toquen módulos en src/api, hooks en src/hooks o lógica de acceso HTTP al backend.


## Arquitectura de frontend y acceso a API – Frontend (modo sugerido: Apply Intelligently)

- No hagas `fetch` directos a URLs hardcodeadas en componentes.
- Todas las llamadas HTTP deben ir a través de:
  - Módulos en `src/api/*`, y/o
  - Hooks en `src/hooks/*` (por ejemplo `use-poa-api`, `use-procedures`, etc.).
- La lógica de headers (`Authorization`, `X-Organization-Id`, `X-Workspace-Id`) debe vivir en la capa de HTTP/contexto, no en los componentes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Core-Business) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
