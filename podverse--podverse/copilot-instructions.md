## app-local-ui-wrappers

> Extract app-local thin wrappers when identical @podverse/ui + i18n configuration repeats (2+ callsites in one app).


# App-local configured UI wrappers

When the **same** configured usage of `@podverse/ui` (including **`next-intl`** strings for `aria-label`, titles, or messages) appears in **two or more** places **within** `apps/web` or **within** `apps/management-web`, extract a thin client component under that app’s `src/components/**` that wires translations and forwards props.

- Keep wrappers **app-local** until **both** apps need the same wiring; then consider `packages/ui` + separate app wrappers for copy (**`shared-ui-i18n`**).
- Do **not** use bare `export { X } from '@podverse/ui'` as a substitute — wrappers must bind meaningful app conventions.

See **`reusable-components`** skill for the full rubric and examples.

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
