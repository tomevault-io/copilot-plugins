## backend-package-nestjs

> All injection tokens MUST be exported as `const` from a dedicated `*.token.ts` file and re-exported from the package's `index.ts`.

# Architectural Rules

## Injection Tokens
All injection tokens MUST be exported as `const` from a dedicated `*.token.ts` file and re-exported from the package's `index.ts`. 
Do NOT use inline strings for injection tokens in modules or providers.

## Logging Format
The development log format is `[App-name@version][lib-name:version][requestId][timestamp][source][libMethod][LEVEL] - message - {payload}`.
- `App-name@version`: Green tag identifying the application (e.g., `[App-example@0.0.3]`).
- `lib-name:version`: Yellow tag identifying the library and its version (e.g., `[@adatechnology/http-client:1.0.0]`).
- `requestId`: Correlation ID from the request context (Cyan).
- `timestamp`: ISO timestamp (Gray).
- `source`: The caller class and method (Magenta, e.g., `[HttpClientController.listPokemon]`).
- `libMethod`: The internal library class and method (Magenta, e.g., `[HttpRedisClient.get]`).
- `level`: Upper-case level (e.g., `INFO`, `DEBUG`) with no extra padding.

When a library logs:
1. It passes its name in the `lib` property.
2. It passes its version in the `libVersion` property.
3. It passes its internal method name in the `libMethod` property.
4. The formatter automatically combines the `context` with `libMethod` to show the full internal path.

Final pattern: `[App][Lib:Version][RequestId][Timestamp][Caller][Lib.Method][LEVEL]`

### Multi-layer Chaining
For complex flows involving multiple libraries or services, the `source` property should act as a breadcrumb of the business logic path, while `libMethod` remains the leaf execution point.
Example for `App -> Service -> Lib`:
- Source: `ServiceA.processData`
- LibMethod: `HttpClient.post`
Output: `[App][Lib][ReqId][Time][ServiceA.processData][HttpClient.post][INFO]`

## Coding Standards
- Prefer `Vanilla CSS` for styling.
- Ensure all packages have consistent token export patterns.
- Keep metadata in logs compact (single line) using `breakLength: Infinity` for `util.inspect`.

---
> Source: [Andersonfrfilho/backend-package-nestjs](https://github.com/Andersonfrfilho/backend-package-nestjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
