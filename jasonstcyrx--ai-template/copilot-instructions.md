## ai-template

> id: use-playwright-scratchpad

id: use-playwright-scratchpad
description: >
  Always demo work using a Playwright script located at `scripts/show-work-script.js`.
  Treat this file as a scratchpad: it may be created, modified, or deleted with each task.
when: 
  - generating code intended for browser-based interaction or UI validation
  - implementing frontend features
  - showcasing functionality through a browser simulation
then: 
  - generate or update `scripts/show-work-script.js` as the single source of Playwright demo code
  - include any setup, navigation, and assertions needed to demonstrate the feature
  - assume this file is temporary and should reflect the most recent task only
  - delete or overwrite any prior content in the file if no longer relevant
  - do not create additional demo scripts unless explicitly instructed


  - During Demo's you may encounter bugs. Either fix them if in scope or log them to gh as known issue. (always ask my permissions first)
  - This is not a replacement for e2e, or unit testing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jasonstcyrx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
