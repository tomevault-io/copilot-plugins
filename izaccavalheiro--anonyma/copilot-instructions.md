## anonyma

> anonyma is a zero-dependency TypeScript library (Node ≥ 18) for PII detection and data anonymization.

# anonyma — Cursor Rules
# These rules are automatically applied by Cursor to every AI interaction in this repository.

## Project Summary

anonyma is a zero-dependency TypeScript library (Node ≥ 18) for PII detection and data anonymization.
It ships:
  - 27 PII detectors (email, phone, SSN, credit card, IBAN, passport, address, VIN, API keys, …)
  - 8 anonymization strategies (mask, redact, pseudonymize, hash/SHA-256, generalize, tokenize, encrypt/AES-GCM, synthesize)
  - 6 compliance presets (GDPR, HIPAA, CCPA, PCI-DSS, SOX, FERPA)
  - Reversible tokenization + LLM pipeline helpers (sanitizeForLLM / restoreFromLLM)
  - WHATWG TransformStream wrappers, batch processing, checksum validators
  - Optional Zod schemas + OpenAI/MCP tool definitions
Published as dual ESM + CJS with full TypeScript declaration files.

---

## Hard Constraints

- ZERO runtime dependencies. Never add to `dependencies` in package.json. zod is an optional peerDependency.
- No `any` type. Always use `unknown` + type guards, generics, or proper unions.
- TypeScript strict mode + `noUncheckedIndexedAccess` + `exactOptionalPropertyTypes` + `noImplicitOverride`. All code must compile clean with `tsc --noEmit`.
- Pure functions for all detectors and non-crypto strategies (no side effects, no global state mutation).
- Immutable inputs — never mutate function arguments. Always return new values.
- Use `.js` file extensions on all relative imports in source (TypeScript NodeNext module resolution).
- JSDoc/TSDoc on every exported symbol: @param, @returns, and at least one @example.

---

## File Structure

```
src/
  index.ts             ← public API barrel — only import point for end users
  types.ts             ← all TS interfaces and type aliases (zero runtime code)
  errors.ts            ← typed error hierarchy rooted at AnonymaError
  anonymize.ts         ← core engine: detect(), anonymize(), anonymizeAsync(), anonymizeObject()
  tokenize.ts          ← tokenize() / detokenize() API
  llm.ts               ← sanitizeForLLM() / restoreFromLLM()
  batch.ts             ← anonymizeBatch(), anonymizeBatchAsync(), detectBatch(), tokenizeBatch()
  presets.ts           ← GDPR/HIPAA/CCPA/PCI-DSS/SOX/FERPA PresetConfig objects
  stream.ts            ← WHATWG TransformStream wrappers (Node ≥ 18 / browser)
  crypto.ts            ← low-level Web Crypto helpers
  validators.ts        ← checksum validators: luhn, verhoeff, nhsMod11, cpfChecksum, …
  schemas.ts           ← optional Zod schemas + AI/MCP tool definitions (do not import outside this file)
  detectors/           ← one file per PII category; each exports detect*() + detect*Aggressive()
    index.ts           ← builds DETECTOR_REGISTRY and AGGRESSIVE_DETECTOR_REGISTRY
  strategies/          ← one file per anonymization strategy
    mask.ts, redact.ts, pseudonymize.ts, hash.ts, generalize.ts, encrypt.ts, synthesize.ts, tokenize.ts
tests/                 ← Vitest test suite mirroring src/
```

---

## Code Style

- `const` by default. `let` only when reassignment is needed.
- `for...of` over `.forEach()` for side-effectful loops.
- Optional chaining `?.` and nullish coalescing `??` over explicit null/undefined checks.
- Validate inputs at the top of functions and throw typed errors; avoid try/catch in hot paths.
- `readonly` on all interface properties that must not be mutated.
- Never use default parameter values that are complex objects — compute them inside the function body.
- Discriminated unions for StrategyOptions (the `strategy` string literal is the discriminant).
- `export type` for type-only exports; `as const` for literal tuples/objects.
- `satisfies` operator to validate objects against types without widening.

---

## Error Handling Rules

- All errors must extend `AnonymaError` from `src/errors.ts`.
- Call `Object.setPrototypeOf(this, new.target.prototype)` in every Error subclass constructor.
- `ValidationError(field, reason)` — for invalid arguments.
- `UnsupportedStrategyError(strategy)` — for unknown strategy names.
- `UnknownCategoryError(category)` — for unknown PII category strings.
- `CryptoNotAvailableError` — when Web Crypto API is absent.
- `EncryptionError` — when AES-GCM fails.
- `PresetNotFoundError(preset)` — when preset name is not in registry.
- Never `throw new Error(...)` directly in library code.

---

## Crypto / Security Rules

- Use Web Crypto API exclusively: `globalThis.crypto.subtle`. Never import `node:crypto`.
- Use `Uint8Array` for binary data. Never use `Buffer`.
- AES-256-GCM for encryption, PBKDF2+SHA-256 (100,000 iterations) for passphrase key derivation.
- Always generate a fresh 12-byte random IV per encrypt() call via `crypto.getRandomValues()`.
- SHA-256 with optional pepper for the hash strategy.
- async only for crypto operations, `Promise.allSettled` for batch async. All other paths must be synchronous.

---

## Adding a New Detector — Checklist

1. Create `src/detectors/<category>.ts`
   - Export `detect<Category>(text: string): PiiMatch[]`
   - Optionally export `detect<Category>Aggressive(text: string): PiiMatch[]`
   - Implement using regex exec loop (not matchAll) to control the global flag object safely
2. Add category string literal to `PiiCategory` union in `src/types.ts`
3. Register in `DETECTOR_REGISTRY` (and `AGGRESSIVE_DETECTOR_REGISTRY`) in `src/detectors/index.ts`
4. Export from `src/detectors/index.ts` barrel
5. Add `TOKEN_PREFIX_MAP` entry in `src/anonymize.ts`
6. Add `TOKEN_PREFIX_MAP` entry in `src/tokenize.ts`
7. Add to `ALL_CATEGORIES` array in `src/anonymize.ts`
8. Write tests in `tests/detectors.test.ts`

---

## Adding a New Strategy — Checklist

1. Create `src/strategies/<strategy>.ts`
   - Export `<strategy>(value: string, options?: <Strategy>Options): string` (sync) or `Promise<string>` (async crypto only)
2. Add strategy string literal to `StrategyName` in `src/types.ts`
3. Add `<Strategy>Options` interface to `src/types.ts`
4. Register in `applyStrategy` / `applyStrategyAsync` switch in `src/anonymize.ts`
5. Re-export from `src/strategies/index.ts`
6. Re-export from `src/index.ts`
7. Write tests in `tests/strategies.test.ts`

---

## Naming Conventions

- Detector functions: `detect<Category>` (camelCase) — e.g. `detectCreditCard`, `detectSsn`
- Aggressive variants: `detect<Category>Aggressive` — e.g. `detectPhoneAggressive`
- Strategy functions: lowercase verb — `mask`, `redact`, `pseudonymize`, `hash`, `generalize`, `encrypt`, `synthesize`
- Error classes: `<Reason>Error` PascalCase — e.g. `ValidationError`, `EncryptionError`
- Compliance preset names: lowercase hyphenated string literals — `"pci-dss"`, `"gdpr"`
- PII category names: lowercase hyphenated — `"credit-card"`, `"date-of-birth"`, `"national-id"`
- Registry constants: SCREAMING_SNAKE_CASE — `DETECTOR_REGISTRY`, `AGGRESSIVE_DETECTOR_REGISTRY`
- Internal unexported helpers: prefix with `_` or leave unexported

---

## Testing Conventions

- Use Vitest globals (`describe`, `it`, `expect`, `beforeEach`) — no imports needed in test files.
- Structure: `describe("<module>") > describe("<function>()") > it("<behaviour description>")`.
- Table-driven tests: use `it.each([[input, expected], ...])`.
- Cover: normal case, empty string, unicode/multibyte, boundary values, invalid inputs, async errors.
- async tests: `async` callback + `await expect(fn()).resolves.toBe(...)` or `rejects.toThrow(...)`.
- Never mock internal modules. Test through the public API surface.
- Coverage gates: 90% lines/functions/statements, 85% branches.

---

## What NEVER to Suggest

- `import ... from "node:crypto"` → use `globalThis.crypto.subtle`
- `Buffer` → use `Uint8Array`
- Adding npm packages for hashing, UUID, string utilities, etc. (zero-dependency constraint)
- `// @ts-ignore` or `// @ts-expect-error` without a documented reason
- `throw new Error(message)` in library code — always use typed subclasses
- `console.log(...)` in source files — only `console.warn` in documented deprecation paths
- Exporting mutable state (arrays, objects) from any module
- `process.env` in source files — this is a library, not an application
- Widening `PiiCategory` to `string`
- Adding `@types/*` to `dependencies` — use `devDependencies`
- `import ... from "zod"` outside `src/schemas.ts`
- `any` type in any form

---

## Subpath Imports

| What you need | Import from |
|---|---|
| Core API (detect, anonymize, tokenize, …) | `"anonyma"` |
| Individual detectors + registry | `"anonyma/detectors"` |
| Zod schemas + AI/MCP tool defs | `"anonyma/schemas"` (requires zod peer) |
| Checksum validators | `"anonyma/validators"` |
| Low-level Web Crypto helpers | `"anonyma/crypto"` |
| TransformStream wrappers | `"anonyma/stream"` |

---

## Compliance Strategy Mapping

| Regulation | Strategy | Categories focus |
|---|---|---|
| GDPR | `pseudonymize` | All personal data identifiers |
| HIPAA | `redact` | 18 PHI Safe Harbor identifiers |
| PCI-DSS | `mask` | credit-card, bank-account, CVV |
| CCPA | `mask` | personal + household data |
| FERPA | `redact` | student education records |
| SOX | `hash` | financial records + employee data |

---

## Useful File References

- Public types → `src/types.ts`
- Error codes → `src/errors.ts`
- Canonical sync detector → `src/detectors/email.ts`
- Canonical sync strategy → `src/strategies/mask.ts`
- Canonical async strategy → `src/strategies/hash.ts` or `src/strategies/encrypt.ts`
- Preset definitions → `src/presets.ts`
- Token format logic → `src/tokenize.ts` + `src/strategies/tokenize.ts`
- LLM integration example → `src/llm.ts`
- Streaming wrappers → `src/stream.ts`
- Validators → `src/validators.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/izaccavalheiro) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
