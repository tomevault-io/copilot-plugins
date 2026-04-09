## planr

> - All components are **standalone** — no NgModules exist or should be created

# CLAUDE.md

## Code conventions

- All components are **standalone** — no NgModules exist or should be created
- Use `inject()` for dependency injection, not constructor injection
- Component selector prefix is `app` (kebab-case); directive prefix is `app` (camelCase) — enforced by ESLint
- New components go under `src/app/features/<feature>/components/` or `src/app/shared/components/`

## Testing

`skipTests: true` is set globally in `angular.json` schematics. Do not generate or create test files unless explicitly asked.

When writing tests, follow the conventions already established in the codebase:

- **Framework**: Karma + Jasmine (run with `npm test`)
- **Services**: use `TestBed` + `provideHttpClient()` + `provideHttpClientTesting()` and `HttpTestingController`
- **Components**: use `TestBed.configureTestingModule({ imports: [TheComponent] })`, set required signal inputs via `fixture.componentRef.setInput()`
- **Mocking**: `jasmine.createSpyObj` for service dependencies; return observables with `of()` or `throwError()`
- **Jasmine types** are available globally via `tsconfig.json` — no import needed for `describe`, `it`, `expect`, etc.

## Commits

Conventional commits are enforced via commitlint. Use `feat:`, `fix:`, `chore:`, etc.

## API

The mock API (`json-server`) must be running on port 3001 for the app to work. Services hardcode `http://localhost:3001` — there is no environment file abstraction.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/franverona)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/franverona)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
