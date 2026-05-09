## impers

> This repository contains `impers`, a Node.js + TypeScript library that wraps libcurl for browser impersonation and HTTP/2/3 support.

# Repository Guidelines

This repository contains `impers`, a Node.js + TypeScript library that wraps libcurl for browser impersonation and HTTP/2/3 support.

## Project Structure & Module Organization
- `src/`: TypeScript source. Key areas include `core/` (Curl handles), `ffi/` (Koffi bindings), `http/` (request/response/session helpers), `websocket/`, `types/`, and `utils/`.
- `tests/`: Jest test suite (e.g., `tests/http.test.ts`) plus shared helpers in `tests/setup.ts` and `tests/mock-server.ts`.
- `examples/`: Runnable usage samples like `examples/simple-get.ts` and `examples/websocket.ts`.
- `dist/`: Generated build output from `npm run build` (do not edit by hand).

## Build, Test, and Development Commands
- `npm run build`: Compile TypeScript from `src/` into `dist/`.
- `npm run lint`: Run ESLint over the codebase.
- `npm test`: Run Jest tests (ESM, `ts-jest` preset).
- `npm run test:watch`: Re-run tests in watch mode during development.

## Coding Style & Naming Conventions

Spritually, the API should be both nodejs native and similar to the sibling python project `curl-cffi`, which you can find in `../curl_cffi`.

However, the API should not be a one-to-one mapping to anything.

- TypeScript with ESM (`"type": "module"`); follow the existing import style and `NodeNext` module resolution.
- Use 2-space indentation, double quotes for strings, and semicolons, matching the current files.
- File names are lowercase and descriptive (`request.ts`, `websocket.ts`). Tests end with `.test.ts`.
- Linting is enforced via ESLint + `typescript-eslint` (`eslint.config.js`); keep code consistent with lint rules.

## Testing Guidelines
- Tests live under `tests/` and are matched by `**/tests/**/*.test.ts`.
- Use `tests/mock-server.ts` for local HTTP behavior when needed.
- Keep new tests focused on observable behavior (status, headers, cookies, body parsing).

## Commit & Pull Request Guidelines
- Commit history is short; existing messages are simple, imperative, sentence-case (e.g., “Add LICENSE”). Follow that style.
- PRs should include: a clear description, tests run (`npm test`/`npm run lint`), and any user-facing API changes (update `README.md` or `examples/` when relevant).

## Environment & Configuration Tips
- Node.js 18+ is required.
- For full fingerprint support, install `curl-impersonate` and set `LIBCURL_PATH` if it is not on the default library path (see `README.md`).

---
> Source: [lexiforest/impers](https://github.com/lexiforest/impers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
