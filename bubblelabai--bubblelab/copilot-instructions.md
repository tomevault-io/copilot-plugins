## api

> For creating tests for the backend in bubblelab-api


Always use pnpm bun test to test backend so it will properly load the setup files
Any shared schemas between front/backend should be written to /packages/bubble-shared-schemas. For the types to show up properly have to run pnpm build:core whenever something changes since it is separate package.
Inspect webhook.test to see how tests are written

---
> Source: [bubblelabai/BubbleLab](https://github.com/bubblelabai/BubbleLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
