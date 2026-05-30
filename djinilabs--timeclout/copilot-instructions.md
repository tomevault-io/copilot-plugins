## ui

> - UI should be split into routes, pages and components.


- UI should be split into routes, pages and components.
- Routes should be configured in the `apps/frontend/src/Routes.tsx` file. Specific routes should go into the `apps/frontend/src/routes` folder.
- components should be React and go into the `apps/frontend/src/components` folder, split by complexity.
- each component should obey the following rules for dependencies:
  - particles should not depend on other internal components
  - atoms should only depend on particles
  - molecules should only depend on atoms or particles
  - higher level components should be placed in an appropriate entity-specific sub-folder like `company`, `unit`, `team`, `team-shifts`, etc.

---
> Source: [djinilabs/timeclout](https://github.com/djinilabs/timeclout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
