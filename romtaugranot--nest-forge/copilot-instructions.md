## nest-forge

> 1. **`index.ts` barrel files** — re-export from every module/folder so imports are clean (e.g. `import { Foo } from './foo'` not `import { Foo } from './foo/foo.service'`).

# Project Conventions

## Imports & Exports

1. **`index.ts` barrel files** — re-export from every module/folder so imports are clean (e.g. `import { Foo } from './foo'` not `import { Foo } from './foo/foo.service'`).
2. **No `import * as`** — always use named imports. If unavoidable, flag it to the user.
3. **Alias ambiguous import names** — alias to something descriptive, e.g. `import { resolve as resolvePath, dirname as getDirectoryName } from 'path'`.
4. **No default exports** — always use named exports. Default exports break barrel files and let import names drift from entity names.
5. **Domain module barrel files expose only the public API** — a domain module's top-level `index.ts` re-exports only what external consumers need: the module class, service(s), and shared types. Schemas, exceptions, refines, and data are implementation details and must not appear in the domain module barrel.
6. **NestJS `exports` array** — a provider (service) that will be injected by another module must appear in the module's `exports` array. Providers used only internally must not be exported.

## Folder Structure

Each entity type lives in its own dedicated folder:

| Entity | Folder | File suffix |
|---|---|---|
| Interfaces | `interfaces/` | `*.interface.ts` |
| Enums | `enums/` | `*.enum.ts` |
| Custom exceptions | `exceptions/` | `*.exception.ts` |
| Type aliases | `types/` | `*.type.ts` |
| Constants | `data/` | `*.data.ts` |
| Utilities | `utils/` | `*.util.ts` |
| Endpoint decorator bundles | `endpoints/` | `*.endpoint.ts` |
| Zod schemas | `schemas/` | `*.schema.ts` |
| Zod refine functions | `refines/` | `*.refine.ts` |
| Transform functions | `transforms/` | `*.transform.ts` |
| DTOs | `dtos/` | `*.dto.ts` |

7. **`common` folder is app-agnostic** — only put code in `common/` that is a pure library with zero knowledge of this application's domain.

## TypeScript Strictness

8. **Always annotate return types** — every function, method, and getter must have an explicit return type annotation. Never rely on inference for return types.
9. **No `any`** — if truly unavoidable, flag it explicitly to the user.
10. **No `as` (type assertions)** — if truly unavoidable, flag it explicitly to the user.
11. **No non-null assertion (`!`)** — handle nullability explicitly. If truly unavoidable, flag it to the user.
12. **`readonly` on immutable properties** — mark interface/class properties that should never be reassigned as `readonly`.
13. **`private readonly` for all constructor injections** — every constructor-injected dependency must be declared `private readonly`.
14. **Prefer `interface` over `type` for object shapes** — use `type` only for unions, intersections, and aliases.

## Code Style

15. **One entity per file** — never define more than one class, interface, or enum in a single file.
16. **`SCREAMING_SNAKE_CASE` for enum keys** — all enum member names must use `SCREAMING_SNAKE_CASE` (e.g. `MY_VALUE`, not `myValue`, `MyValue`, or `my-value`).
17. **Custom exceptions only** — always define and throw domain-specific exception classes; never throw plain `Error` or generic built-ins.
18. **Constants in `data/` only — one concern per file** — never hardcode constant values inline in logic or service files. Each `*.data.ts` file must contain constants that belong to a single cohesive topic, and the file name must precisely reflect that topic (e.g. `app-config-path.data.ts` holds only the config path constant). Never group unrelated constants in one file because they happen to share a module or prefix.
19. **Use destructuring** — use object and array destructuring when appropriate instead of repeated property access.
20. **Prefer getters over methods for read access — place them last** — whenever a method only reads and returns a value (no side effects, no parameters), use a `get` accessor instead. All `get` accessors must be placed after all other methods, at the bottom of the class body.
21. **No `function` keyword** — always use arrow functions (`const foo = () => {}`). Never use `function` declarations or expressions.
22. **No nested ternaries** — a ternary inside a ternary must be an `if/else` block. Keep ternaries flat and simple.
23. **`const` over `let` by default** — only use `let` when reassignment is genuinely needed.
24. **No reserved words as identifiers** — never use a JavaScript/TypeScript reserved word (e.g. `type`, `class`, `default`, `delete`, `in`, `new`, `return`, `throw`) as a variable, constant, or property name. Use a descriptive, domain-specific alternative instead (e.g. `credentialType` instead of `type`).
25. **No abbreviations** — always spell out identifiers in full. Never abbreviate (e.g. `error` not `err`, `config` not `cfg`, `response` not `res`, `request` not `req`). Only use abbreviations that are universally established conventions (e.g. `id`, `url`, `ssh`).
56. **No one-line `if` statements** — always use a block body (`{ }`) for `if`, `else if`, and `else` branches, even when the body is a single statement. Never write `if (condition) doSomething();` on one line.
26. **Two-step read-then-parse for file I/O** — when reading and parsing a file (e.g. JSON), use two separate `try/catch` blocks with the intermediate variable declared with `let` before both. This ensures each failure — read error vs. parse error — maps to its own distinct exception class. Never place both operations inside a single `try` block.

## Configuration Files

27. **Config files must be JSON** — all configuration files must use the `.json` format. No YAML, TOML, `.env`-only, or other formats for config.
28. **Config files must use `camelCase` keys** — every key at every level must be `camelCase`. No `snake_case`, `kebab-case`, `PascalCase`, or `SCREAMING_SNAKE_CASE` keys.
29. **Config files must be hierarchically structured** — related settings must be grouped under a named parent key rather than flattened at the root level. Flat lists of unrelated top-level keys are not acceptable.
30. **Always access config via a getter — never call `configService` directly at the usage site** — every value read from `configService` must be wrapped in a `get` accessor on the enclosing class. The getter calls `configService.getOrThrow` or `configService.get` internally; nothing outside that getter may call `configService` directly for that value.
31. **Inject `ConfigService` without a generic type parameter** — declare it as `ConfigService`, not `ConfigService<AppConfig>` or `ConfigService<AppConfig, true>`. Use explicit return type annotations on the getter instead of relying on the generic to infer types. Never pass `{ infer: true }` as an argument to `getOrThrow`.
32. **Use `getOrThrow` with the deepest dot-notation key path — store it as a constant** — always call `configService.getOrThrow('x.y.z')` rather than `configService.getOrThrow('x').y.z`. If two getters both need values under `x.y`, each still calls `getOrThrow` with its own full path. Every key path string must be extracted into a named constant in the relevant `*.data.ts` file — never hardcode a path inline inside a getter.
33. **Use `configService.get` for optional config values** — when a config value may legitimately be absent, use `configService.get` instead of `getOrThrow`. Annotate the getter's return type as `T | undefined` (or `T | null | undefined` if the schema allows null). The same key-path-as-constant rule applies.
34. **Config key path constant naming** — constants for config key paths follow two patterns: `[DOMAIN]_CONFIG_PATH` for the root segment (e.g. `CREDENTIALS_CONFIG_PATH = 'credentials'`) and `[DOMAIN]_[FIELD]_CONFIG_KEY` for the full dot-notation path composed from it (e.g. `` CREDENTIALS_BASE_PATH_CONFIG_KEY = `${CREDENTIALS_CONFIG_PATH}.basePath` ``).
35. **Use the Zod-inferred type as the return type of config getters** — when a config getter returns an object read from `configService`, its explicit return type annotation must be the `z.infer<...>` type derived from the corresponding Zod schema, not a manually written interface or `any`.

## Validation & Exceptions

36. **Validate northbound requests and responses with Zod** — every incoming request and outgoing response on the northbound (API-facing) side must be validated through a Zod schema.
37. **Validate and transform southbound responses with Zod** — every response received from a southbound dependency (external service, DB, etc.) must be validated with a Zod schema. Additionally, transform any properties that don't follow `camelCase` or that use unclear naming to conform to both conventions before the data propagates inward.
38. **`.describe()` on northbound Zod schema fields** — add `.describe('...')` to every field in schemas that appear on the northbound (API-facing) surface. These descriptions are picked up by `nestjs-zod` for automatic Swagger documentation. Never manually annotate fields with `@ApiProperty` when the field already has a Zod schema.
39. **Custom exceptions must extend a NestJS `HttpException`** — never create a custom exception class that does not extend one of NestJS's built-in `HttpException` subclasses (e.g. `BadRequestException`, `NotFoundException`, `UnauthorizedException`).
40. **Never set `this.name` in custom exceptions** — do not assign `this.name` inside a custom exception constructor. NestJS `HttpException` subclasses handle identity through the class itself; setting `this.name` is redundant and inconsistent.
41. **One exception class per distinct error — messages belong inside the class — pass the raw `Error`** — never instantiate a custom exception with a message string passed from the call site. If two throws produce different messages, they must be two different exception classes, each owning its message internally. When catching and rethrowing, always pass the original `Error` object as a constructor parameter — never extract `.message` or parse the error at the call site. Only truly dynamic runtime values (e.g. an OS error cause, a Zod error detail) may be passed as constructor parameters to fill a template; the template itself still lives inside the class.
42. **Split Zod entities across `schemas/`, `types/`, and `dtos/`** — never mix these in one folder. Zod schema definitions go in `schemas/`, inferred TypeScript types (`z.infer<...>`) go in `types/`, and NestJS DTO classes go in `dtos/`.
43. **Use a real TypeScript enum with `z.enum` — pass the enum directly** — whenever using `z.enum(...)` for Zod validation, always define a proper TypeScript `enum` (in an `enums/` file) and pass the enum object directly to `z.enum` (e.g. `z.enum(MyEnum)`).
44. **No inline literals in Zod schemas — all validation constants go in a `data/` file** — never write raw strings, numbers, regexes, or default values directly inside a schema definition. Every such value (e.g. min/max lengths, allowed patterns, default values) must be extracted into a named constant in the relevant `*.data.ts` file and referenced from there.
45. **Always use `z.safeParse` — never `z.parse`** — always use `safeParse` for runtime Zod validation. `parse` throws a raw `ZodError` which bypasses custom exception classes and the exception filter. `safeParse` forces explicit handling of the failure path, where a domain exception is thrown.
46. **`z.discriminatedUnion` for tagged unions** — when modeling a union of object schemas that share a common literal discriminant field, always use `z.discriminatedUnion('fieldName', [...])` instead of `z.union`. It produces more precise TypeScript types and clearer validation error messages.
54. **Two-schema pattern for non-`camelCase` southbound data** — when a southbound source (external API, HTTP headers, DB row, etc.) returns keys that are not `camelCase`, define two schemas in the same file. The first, named with a `Raw` suffix (e.g. `RequestIdHeadersRawSchema`), validates the wire format exactly as received. The second (e.g. `RequestIdHeadersSchema`) is derived from the first via `.transform()` and remaps every non-`camelCase` key to a `camelCase` equivalent. Only the transformed schema may be used outside the schema file; the raw schema is an implementation detail of the transform and must not be imported elsewhere.

## Controllers

47. **Controllers are pure routers — one service call, no logic** — a controller method must do nothing except call the relevant service method and return its result. It must not access the request object, manipulate the response object, transform data, branch on conditions, or perform any business logic. All of that belongs in the service.
52. **Extract endpoint decorator bundles into `endpoints/`** — all per-method decorators must be composed with `applyDecorators` and exported as a named constant from a `*.endpoint.ts` file under an `endpoints/` folder. Each bundle function accepts three parameters: the NestJS HTTP method factory (e.g. `Post`, `Get`, `Delete`), the route string (e.g. `':id'`, `'run'`, `''` for root), and the success `HttpStatus` code. The bundle applies `httpMethod(route)`, `HttpCode(successStatus)`, and all metadata decorators (`@ZodSerializerDto`, `@ApiOperation`, `@ApiResponse`, etc.) internally. The controller method applies the bundle as a single decorator call (e.g. `@CreateSessionEndpoint(Post, '', HttpStatus.CREATED)`) with no other decorators on the method.

## Async

48. **Always `await` every async call — never let promises hang** — every call that returns a `Promise` must be explicitly `await`ed, even when returning the result from a function. For example, when mapping an array to promises, always make the callback `async` and `await` the call inside it rather than returning the bare promise.
55. **No `.then()` or `.catch()` — use `async`/`await` only** — never chain `.then()` or `.catch()` on a promise. Use `async`/`await` with `try`/`catch` instead. The only exception is when you genuinely have no access to an `async` context and cannot create one (e.g. a top-level module initialisation expression) — flag it explicitly if so.

## NestJS

49. **Apply `ZodValidationPipe` globally — never per-endpoint** — register `ZodValidationPipe` as a global pipe in `main.ts` (`app.useGlobalPipes(new ZodValidationPipe())`). Never pass it as an argument to individual `@Body`, `@Param`, `@Query`, or other parameter decorators.
51. **Never pluck a single property from `@Query`, `@Param`, or similar — always consume the full object** — never pass a property name to these decorators (e.g. `@Query('state')` or `@Param('id')`). Always use the bare decorator (`@Query()`, `@Param()`) to receive the full object, typed and validated against a dedicated Zod DTO. This ensures all inputs are schema-validated and keeps the controller signature honest about what it receives.
50. **Always serialize responses with `@ZodSerializerDto`** — every controller method must have a `@ZodSerializerDto(ResponseDto)` decorator (from `nestjs-zod`) pointing to the appropriate response DTO, so outgoing responses are validated and stripped through the Zod schema before being sent to the client.
53. **Avoid injecting the raw request or response — flag it when unavoidable** — never use `@Req`, `@Res`, `@Request`, `@Response`, or any other mechanism that exposes the raw HTTP request or response object, in either a controller or a service. Use purpose-built NestJS decorators (`@Headers`, `@Ip`, etc.) or dedicated interceptors/middleware to extract what you need instead. If injecting the raw request or response is truly unavoidable, flag it explicitly to the user with a comment explaining why no alternative exists.

---
> Source: [romtaugranot/nest-forge](https://github.com/romtaugranot/nest-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
