## frontend

> React/Vite/Material-UI frontend guidelines for Ship Status Dashboard


* After making changes, always run formatting and linting to maintain consistency:

```bash
cd frontend && npx eslint . --fix && npx prettier --write .
```

* Prefer functional components and React hooks over class components.
* Keep UI elements consistent with Material-UI standards.
* Use MUI's `styled()` for custom component styling — never use inline `sx` for non-trivial styles. Follow these patterns:
  - Import from `@mui/material`: `import { styled } from '@mui/material'`
  - Wrap MUI components: `const StyledCard = styled(Card)(({ theme }) => ({ ... }))`
  - Wrap HTML elements: `const Logo = styled('img')(({ theme }) => ({ ... }))`
  - Custom props with TypeScript generics: `const StatusChip = styled(Chip)<{ status: string }>(({ theme, status }) => ({ ... }))`
  - Always use theme values (`theme.palette`, `theme.spacing()`, `theme.breakpoints`) instead of hardcoded colors or sizes.
* The frontend uses `npm`. If you must install or update any dependencies, always use the `--ignore-scripts` flag.
* Environment variables use the `VITE_` prefix (e.g. `VITE_PUBLIC_DOMAIN`, `VITE_PROTECTED_DOMAIN`).
* When adding or changing a React Router route, also update the `metaRoutes` patterns in `cmd/dashboard/meta.go` so the server-side Open Graph metadata injection stays in sync. This drives link previews in Slack and other clients that read OG tags.
* Team SLO workspace UI is selected in `frontend/src/components/team/slo/registry.tsx`. Versioned renderers live under `frontend/src/components/team/slo/{team}/v{n}/` (TRT `payload_streams` v1 is `trt/v1/`). Unknown `(kind, schema_version)` pairs use `UnknownSLOWorkspace`. Add and edit controls render only when `isTeamSLOAdmin(team)` is true.

---
> Source: [openshift-eng/ship-status-dash](https://github.com/openshift-eng/ship-status-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-08 -->
