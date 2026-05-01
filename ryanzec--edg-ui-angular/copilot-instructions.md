## edg-ui-angular

> - **NEVER** use `Directive` in the name of the selector

# General Angular Directive Patterns
- **NEVER** use `Directive` in the name of the selector
- **NEVER** use `standalone: true` in the `@Directive` decorator.
- **ALWAYS** have a export default value for each input of the directive using the pattern of `{DIRECTROY_NAME - Directive part}_{INPUT_NAME - selector naming part}_DEFAULT`.
- **ALWAYS** make sure sure all inputs and outputs of a directive are prefix with the camelCase version of the directive's class name minus the `Directive` part like this:
```ts
// ...
export class ScrollAreaDirective {
  // ...
  public scrollAreaDirection = input<ScrollAreaDirection | ''>(SCROLL_AREA_DIRECTION_DEFAULT);
  public scrollAreaOnlyShowOnHover = input<boolean>(SCROLL_AREA_ONLY_SHOW_ON_HOVER_DEFAULT);
  // ... 
```

---
> Source: [ryanzec/edg-ui-angular](https://github.com/ryanzec/edg-ui-angular) — distributed by [TomeVault](https://tomevault.io/claim/ryanzec).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
