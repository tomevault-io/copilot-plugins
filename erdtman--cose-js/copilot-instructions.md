## cose-js

> cose-js is a JavaScript implementation of CBOR Object Signing and Encryption (COSE), [RFC 9052](https://datatracker.ietf.org/doc/html/rfc9052).

# CLAUDE.md

## Project overview

cose-js is a JavaScript implementation of CBOR Object Signing and Encryption (COSE), [RFC 9052](https://datatracker.ietf.org/doc/html/rfc9052).

## Setup

The test suite depends on a git submodule at `test/Examples` (from https://github.com/cose-wg/Examples). You must initialize it before running tests:

```sh
git submodule update --init
```

## Common commands

- **Lint:** `npx semistandard`
- **Test:** `npm test`
- **Lint + Test:** `npm test` (runs lint automatically via `pretest`)

## Code style

- Follows [semistandard](https://github.com/standard/semistandard) (standard style with semicolons).
- Use `async/await` over raw promises.

---
> Source: [erdtman/cose-js](https://github.com/erdtman/cose-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
