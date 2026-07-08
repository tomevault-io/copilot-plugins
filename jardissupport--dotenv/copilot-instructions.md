## dotenv

> `.env` loader with two modes (Public + Private), two-stage `APP_ENV` bootstrap, cascade includes (`load()`/`load?()`), `${VAR}`/`~` substitution via `VariableRegistry`, `_FILE` secret resolution, and cast chain (Value → UserHome → Numeric → Bool → JSON → Array).

# jardissupport/dotenv

`.env` loader with two modes (Public + Private), two-stage `APP_ENV` bootstrap, cascade includes (`load()`/`load?()`), `${VAR}`/`~` substitution via `VariableRegistry`, `_FILE` secret resolution, and cast chain (Value → UserHome → Numeric → Bool → JSON → Array).

## Usage essentials

- **`loadPublic($path)` vs. `loadPrivate($path)`:** Public writes `putenv()` + `$_ENV` + `$_SERVER` (bootstrap, once per request) and returns `void`. Private returns `array<string,mixed>` without globals — this is the default for domain configs (`Infrastructure/Config/*Config` classes). Inject values as primitives into the domain, never inject the `DotEnv` service itself.
- **Two-stage bootstrap is fixed:** Stage 1 loads `.env` + `.env.local`, then `APP_ENV` is resolved from `VariableRegistry`/`$_ENV`/`getenv()`, Stage 2 loads `.env.{APP_ENV}` + `.env.{APP_ENV}.local`. Later files override earlier ones — `*.local` always comes after the base/env counterpart.
- **Cast chain runs in strict order with early exit on non-string:** `CastStringToValue` → `CastUserHome` → `CastStringToNumeric` → `CastStringToBool` → `CastStringToJson` → `CastStringToArray`. Add custom handlers via `DotEnv::addHandler($invokable, prepend: true)` before substitution; never call `CastTypeHandler` directly. Note: `ENABLED=1` becomes `int(1)` (Numeric takes precedence over Bool) — write `true`/`false` explicitly for booleans.
- **`VariableRegistry` is the single source of truth** for `${VAR}` and `~` expansion in both modes; `LoadValuesFromFiles` populates it before every cast. Never use `getenv()` directly for values from `.env` in code — otherwise Private mode isolation does not apply.
- **Include system:** `load(path.env)` is required (throws `EnvFileNotFoundException`), `load?(path.env)` is optional (silent skip); relative paths are resolved from the directory of the including file; each include runs the full cascade (base → .local → .{APP_ENV} → .{APP_ENV}.local). Circular includes are detected via a `realpath()` stack and throw `CircularEnvIncludeException::getIncludeStack()`.
- **`_FILE` pattern + optional `jardissupport/secret`:** Keys with the `_FILE` suffix (`DB_PASSWORD_FILE=/run/secrets/db_pw`) are read by the loader, trimmed, passed through the cast chain, and stored under the key without the suffix (`DB_PASSWORD`). Combinable with `jardissupport/secret`: if the file contains `secret(aes:...)` it is decrypted in the same pass. Layer rule: `DotEnv` lives in `Infrastructure`, **never** in the domain.

## Full reference

https://docs.jardis.io/en/support/dotenv

---
> Source: [jardisSupport/dotenv](https://github.com/jardisSupport/dotenv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
