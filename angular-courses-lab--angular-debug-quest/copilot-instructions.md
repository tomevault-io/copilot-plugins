## angular

> Global Angular application rules


# Angular v19 Cursor rules

- use `declarative` over `imperative` pattern
- always add type signature to functions

## Javascript

- prefer `const` over `let` when possible

## Typescript

- use `type` over `interface`
- if a type is used in a single file, declare it at the top of the same file, under the imports section. Otherwise create a a dedicated `*.model.ts` file.
- do not prefix types with `I`

## Dependancy Injection

- use `inject()` over `contructor`

## Components

- do not specify `standalone: true`
- do not add empty Angular lifecycle hooks
- do not include a `.css` file
- use `afterNextRender` over `afterViewInit`
- use `[class]` over `[ngClass]`

## Services

- do not create a service if only ment for functions without a shared state or DI. Prefer utility functions in a `*.utils.ts` file.

## Rxjs

- on component destruction, unsubscribe to observables meant to query data (such as a state or a GET endpoint), not to change it (such as a POST, PUT or DELETE endpoints).
- do not nest subscriptions
- use `subscribe()` callback rather than `tap()` to update local state if `.subscribe()` is used
- use the `async` pipe over a `.subscribe()` in components for data meant to be used in the template only
- do not use `.subscribe()` in services, expose an Observable for the consumer to subscribe

## Routing

- lazy load each route
- use `withComponentInputBinding`

## Styling

- use TailwindCSS whenever it's possible
- do not use any other UI library
- use css, not scss or less

## Third-party libraries

- only use libraries defined in [package.json](mdc:package.json)

## Formatting

- follow [.prettierrc.json](mdc:.prettierrc.json) rules

## Coding rules

### named parameters pattern

Follow the `named parameters` pattern for parameters.

```ts
// DO NOT
function getScripts(searchTerm: string; limit: number) {}

// DO
function getScripts(options: {searchTerm: string; limit: number}) {}
```

---
> Source: [angular-courses-lab/angular-debug-quest](https://github.com/angular-courses-lab/angular-debug-quest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
