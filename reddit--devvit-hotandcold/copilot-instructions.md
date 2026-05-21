## client

> Use when writing webview code


Guidelines:

- Use NPM dependencies if needed, ensure they are web compatible
- You cannot use websockets. Call devvit_search and query for "realtime" to get more information
- Ensure you are using "preact" instead of "react"
- Prefer the use of preact signal over hooks
- When testing, target the specific file with this command: npm run test:client -- guess.test.ts

---
> Source: [reddit/devvit-HotAndCold](https://github.com/reddit/devvit-HotAndCold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
