## pfwc

> This repository contains an exploratory Wagmi connector and a draft specification for Protocol for Wallet Connection (PFWC). The connector demo lives in `demo/` and uses txlink's current stored-request API; draft protocol behavior is not yet implemented by txlink.

# PFWC Specification

## Project

This repository contains an exploratory Wagmi connector and a draft specification for Protocol for Wallet Connection (PFWC). The connector demo lives in `demo/` and uses txlink's current stored-request API; draft protocol behavior is not yet implemented by txlink.

## Before Making Changes

- Read `docs/specification.md` and `docs/implementation-notes.md` before changing protocol behavior.
- Read `demo/README.md` and `demo/src/txlinkConnector.ts` before changing the proof-of-concept connector.
- Check `docs/security-considerations.md` when changing capability, visibility, authentication, expiry, or error behavior.
- Keep `README.md` and `docs/examples.md` synchronized with the normative specification.

## Specification Rules

- Use the key words MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, NOT RECOMMENDED, MAY, and OPTIONAL only as defined by RFC 2119 and RFC 8174.
- Preserve wire identifiers exactly, including `chainId`, `account`, `method`, `params`, `requestUrl`, `uri`, and `expiresAt`.
- Treat chain and account identifiers as opaque strings. Examples may use EVM identifiers but must not make EVM behavior normative.
- Prefer the smallest interoperable contract. Do not add optional aliases or backward-compatibility fields without a concrete deployed requirement.
- Update `docs/implementation-notes.md` before committing protocol changes.

## Documentation

- Keep examples valid JSON and use complete HTTP methods, paths, headers, and status codes.
- Never place a completion token in a URL query parameter. Capability tokens belong in the URI fragment and the `Authorization` header only.
- Assume all repository content will be public. Do not include personal information or real secrets.

## Demo

- Use Bun for package management and scripts.
- Use Wagmi and viem for wallet behavior, React Query for async React state, and Zod for external input validation.
- Keep the connector transport isolated in `demo/src/txlinkConnector.ts`.
- Run `bun run format`, `bun run lint`, `bun test`, and `bun run build` from `demo/` after TypeScript or React changes.

---
> Source: [stephancill/pfwc](https://github.com/stephancill/pfwc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
