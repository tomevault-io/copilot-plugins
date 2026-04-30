## angular-tailwind-template

> You are an expert in TypeScript, Angular, and scalable web application development. You write functional, maintainable, performant, and accessible code following Angular and TypeScript best practices.


You are an expert in TypeScript, Angular, and scalable web application development. You write functional, maintainable, performant, and accessible code following Angular and TypeScript best practices.

## TypeScript Best Practices

- Use strict type checking
- Prefer type inference when the type is obvious
- Avoid the `any` type; use `unknown` when type is uncertain

## Angular Best Practices

- Always use standalone components over NgModules
- Must NOT set `standalone: true` inside Angular decorators. It's the default in Angular v20+.
- Use signals for state management
- Implement lazy loading for feature routes
- Do NOT use the `@HostBinding` and `@HostListener` decorators. Put host bindings inside the `host` object of the `@Component` or `@Directive` decorator instead
- Use `NgOptimizedImage` for all static images.
  - `NgOptimizedImage` does not work for inline base64 images.

## Accessibility Requirements

- It MUST pass all AXE checks.
- It MUST follow all WCAG AA minimums, including focus management, color contrast, and ARIA attributes.

### Components

- Keep components small and focused on a single responsibility
- Use `input()` and `output()` functions instead of decorators
- Use `computed()` for derived state
- Set `changeDetection: ChangeDetectionStrategy.OnPush` in `@Component` decorator
- Prefer inline templates for small components
- Prefer Reactive forms instead of Template-driven ones
- Do NOT use `ngClass`, use `class` bindings instead
- Do NOT use `ngStyle`, use `style` bindings instead
- When using external templates/styles, use paths relative to the component TS file.

## State Management

- Use signals for local component state
- Use `computed()` for derived state
- Keep state transformations pure and predictable
- Do NOT use `mutate` on signals, use `update` or `set` instead

## Templates

- Keep templates simple and avoid complex logic
- Use native control flow (`@if`, `@for`, `@switch`) instead of `*ngIf`, `*ngFor`, `*ngSwitch`
- Use the async pipe to handle observables
- Do not assume globals like (`new Date()`) are available.
- Do not write arrow functions in templates (they are not supported).

## Services

- Design services around a single responsibility
- Use the `providedIn: 'root'` option for singleton services
- Use the `inject()` function instead of constructor injection

## Typescript

- NEVER use type assertions. Eg: `const x = value as Type` is forbidden. The only exception is `as const`.
- NEVER use `any` type.
- NEVER use the not null assertion operator (`!`). Prefers the optional chaining operator (`?.`) or explicit checks.
- Use type narrowing to check for not null or undefined values, such as `if (value !== null && value !== undefined)`.
- Use `===` instead of `==`.
- Define return types explicitly for functions.
- Use `const` for variables that are not reassigned, and `let` for those that are.
- When working with union types, use `never` to ensure that if new cases are added, TypeScript will throw an error if they are not handled.
- Avoid type checking with `instanceof`. Always prefer to specify types of variables, parameters and return values to leverage the full power of TypeScript features.
- Prefer immutability with `readonly`.

## TailwindCSS V4 Best Practices

- `text-opacity-*` → Use `text-{color}/{opacity}` instead
- `bg-opacity-*` → Use `bg-{color}/{opacity}` instead
- `border-opacity-*` → Use `border-{color}/{opacity}` instead
- `flex-grow-*` → Use `grow-*`
- `flex-shrink-*` → Use `shrink-*`
- `decoration-slice` → Use `box-decoration-slice`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MatthewDlr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
