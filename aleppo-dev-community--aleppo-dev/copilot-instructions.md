## front-end

> - server components for fetching (or tanstack/query in case of client requests)

## rules
- server components for fetching (or tanstack/query in case of client requests)
- never use useEffect for data fetching
- create separate pages for forms (don't use dialogs), only exceptions is delete or simliar actions dialogs
- use tanstack query mutations (useMutation) for actions.
- don't use `any` types
- use named export for anything except nextjs pages and layouts
- use alias import when possible

---
> Source: [aleppo-dev-community/aleppo.dev](https://github.com/aleppo-dev-community/aleppo.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
