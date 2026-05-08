## 100-monorepo

> - CLI (`gitnexus/`): `npm test`; `npm run test:integration`; `npx tsc --noEmit`.


# GitNexus build/test quick refs

- CLI (`gitnexus/`): `npm test`; `npm run test:integration`; `npx tsc --noEmit`.
- Web (`gitnexus-web/`): `npm test`; `npm run dev`; `npx tsc -b --noEmit`; `E2E=1 npx playwright test` (needs servers).
- `npm install` in `gitnexus/` runs `prepare` (tsc build) and `postinstall` (tree-sitter patches); needs `python3`, `make`, `g++`.
- LadybugDB locking tests may fail in containerized environments because of `/tmp` file locks (known issue, not a code bug).

---
> Source: [abhigyanpatwari/GitNexus](https://github.com/abhigyanpatwari/GitNexus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
