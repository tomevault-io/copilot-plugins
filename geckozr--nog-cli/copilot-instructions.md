## nog-cli

> Operational instructions for AI assistants working in this repository. Keep it short, keep it actionable. If you are a human contributor, see [CONTRIBUTING.md](./CONTRIBUTING.md) and [docs/DOCUMENTATION.md](./docs/DOCUMENTATION.md).

# CLAUDE.md

Operational instructions for AI assistants working in this repository. Keep it short, keep it actionable. If you are a human contributor, see [CONTRIBUTING.md](./CONTRIBUTING.md) and [docs/DOCUMENTATION.md](./docs/DOCUMENTATION.md).

## What is nog-cli

A CLI that generates a strict, type-safe NestJS SDK from an OpenAPI v3 specification. Output: DTO classes annotated with `class-validator`, NestJS service classes with dual-method endpoints (Observable by default, Promise variant via `Async` suffix), an `ApiModule` with `forRoot` / `forRootAsync`, and barrel `index.ts` files.

## Pipeline

`OpenAPI doc -> Parser -> IR -> Generator (writers + writers/core builders) -> Prettier`.

- **Parser** (`src/core/parser/`): loads + bundles the spec via `swagger-parser`.
- **IR** (`src/core/ir/`): the framework-agnostic intermediate representation. **Business logic lives here**, not in the generator or the parser.
- **Generator** (`src/core/generator/`): orchestrator (`engine.ts`) calls each `*.writer.ts`, which composes AST nodes through the reusable builders in `writers/core/*` and finally serialises via `ts.Printer` + Prettier.

See [docs/DOCUMENTATION.md](./docs/DOCUMENTATION.md) for the architectural deep dive.

## Development commands

| Command                     | What it does                                                                                         |
| --------------------------- | ---------------------------------------------------------------------------------------------------- |
| `npm run build`             | TS production build (`tsc -p tsconfig.build.json`).                                                  |
| `npm test`                  | Vitest unit suite (`test/units`).                                                                    |
| `npm run test:e2e`          | E2E generation suite (real-world + complex fixtures, validates AST via the TypeScript Compiler API). |
| `npm run test:all`          | Units + E2E in one run.                                                                              |
| `npm run test:coverage`     | Coverage report.                                                                                     |
| `npm run lint` / `lint:fix` | ESLint.                                                                                              |
| `npm run format`            | Prettier over `src/` and `test/`.                                                                    |
| `npm run deps:check`        | Run before adding any new import to avoid stale dependencies.                                        |
| `npm run docs`              | TypeDoc developer docs into `dist-docs/`.                                                            |

Husky hooks: `pre-commit` runs `lint-staged`, `commit-msg` runs `commitlint`. Never bypass them (`--no-verify` is forbidden).

## Hard constraints

These rules are non-negotiable.

- **AST over strings.** All generated code MUST flow through `ts.factory.create*` and the builders in `src/core/generator/writers/core/*` (`DecoratorBuilder`, `PropertyBuilder`, `TypeBuilder`, etc.). Never use string concatenation or template literals to assemble TypeScript source.
- **DI for writers.** Every writer receives its builders through the constructor. Keep them stateless and testable; new dependencies go in via DI, never as module-level singletons.
- **No `any`.** Use `unknown` plus narrowing, or define a specific interface. Prefix legitimately unused parameters with `_`.
- **No `console.*`** in feature code. Use the static `Logger` in `src/utils/logger.ts`.
- **English-only** comments, docs, commit messages, and runtime output.
- **No emoji** in comments, commit messages, generated code, runtime output, or documentation.
- **Conventional Commits** (`feat:`, `fix:`, `chore:`, `refactor:`, `test:`, `docs:`). `commitlint` will reject anything else.
- **Never `--no-verify`** on `git commit` / `git push`. If a hook fails, fix the underlying issue.
- **Meaningful comments only.** Explain _why_, not _what_. Delete a comment if removing it would not confuse a future reader.

## Where things live

```
src/
  cli/                                   # commander wiring, CLI entry point
    commands/                            #   generate command, etc.
  config/                                # config.loader, CLI flags merge
  core/
    parser/                              # OpenAPI -> Document
    ir/                                  # IR types, analyzer, converter (business logic)
      interfaces/models.ts               #   IR type definitions
      analyzer/                          #   type/validator mappers, schema merger
      openapi.converter.ts               #   OpenAPI -> IR transformer
    generator/
      engine.ts                          # writer orchestrator (filesystem writes)
      helpers/type.helper.ts             # shared naming / type-name utilities
      writers/                           # one *.writer.ts per generated file type
        core/                            #   reusable AST builders (12 of them)
  utils/                                 # Logger, naming, FS helpers
test/
  units/                                 # Vitest, mock dependencies, camelCase mocks
  e2e/                                   # Vitest, fixture-driven, validates AST via ts.createSourceFile
  fixtures/                              # OpenAPI spec fixtures (standard, real-world, edge-case)
```

## Testing strategy

- **Unit tests** (`test/units/`) mock dependencies with strict camelCase naming (e.g. `decoratorBuilderMock`, `dtoWriterMock`). Isolate the unit under test; do not reach into other writers.
- **E2E tests** (`test/e2e/`) generate real code from fixtures and validate the result structurally with `ts.createSourceFile` + visitor traversal, not by file existence. When you add a writer feature, add a structural assertion that proves the new AST shape lands in the output.
- Coverage target: greater than 90% line coverage overall, 100% on critical paths.

## Non-obvious gotchas

These are the things that are easy to break and not obvious from the code:

- **Dual service methods.** Every endpoint produces two overloads from a single IR operation: an Observable (default) and a Promise variant (with `Async` suffix). Keep both in sync in `ServiceWriter`; never emit only one.
- **Dynamic `class-validator` imports.** The DTO writer tracks which validator decorators it actually emits and only adds them to the import statement. Adding a new validator means extending `VALIDATOR_DECORATOR_MAP` _and_ making sure the writer adds the name to the import set.
- **`oneOf` shapes.** A schema with only `oneOf` becomes a pure union type alias; `oneOf` combined with sibling properties becomes an intersection with a base class. See the IR analyzer.
- **Headers vs query parameters** are currently collapsed in the IR; the open TODOs in `parameter-builder.ts` and `service.writer.ts` describe the planned split. If you touch parameter generation, read those TODOs first.
- **File uploads.** `ServiceWriter` flips a `serviceUsesFileUploads` marker that drives the import of multipart utilities. Do not bypass it.
- **Generated module name** is always `api.module.ts`. Header generation (`HeaderGenerator`) stamps every generated file with CLI version + spec version — keep both in sync if you change the header.

## Documentation surface

The `docs/` folder is versioned and shipped to users:

- [docs/DOCUMENTATION.md](./docs/DOCUMENTATION.md) is the canonical architectural reference and is consumed by TypeDoc as the developer-guide landing page.
- [docs/USAGE.md](./docs/USAGE.md) is end-user documentation rendered into generated SDKs.

When you change architecture, update `docs/DOCUMENTATION.md`. When you change the CLI surface, update [README.md](./README.md). When you change contributor workflow, update [CONTRIBUTING.md](./CONTRIBUTING.md).

## Security

Vulnerabilities go through GitHub's Private Vulnerability Reporting (Security tab). Do not edit [SECURITY.md](./SECURITY.md) without review.

---
> Source: [geckozr/nog-cli](https://github.com/geckozr/nog-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
