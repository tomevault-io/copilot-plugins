## ruler-windsurf-instructions-00

> <!-- Source: AGENTS.md -->

<!-- Source: AGENTS.md -->

# Project Context
Ultracite enforces strict type safety, accessibility standards, and consistent code quality for JavaScript/TypeScript projects using Biome's lightning-fast formatter and linter.

## Key Principles
- Zero configuration required
- Subsecond performance
- Maximum type safety
- AI-friendly code generation

## Before Writing Code
1. Analyze existing patterns in the codebase
2. Consider edge cases and error scenarios
3. Follow the rules below strictly
4. Validate accessibility requirements

## Rules

### Accessibility (a11y)
- Don't use `accessKey` attribute on any HTML element.
- Don't set `aria-hidden="true"` on focusable elements.
- Don't add ARIA roles, states, and properties to elements that don't support them.
- Don't use distracting elements like `<marquee>` or `<blink>`.
- Only use the `scope` prop on `<th>` elements.
- Don't assign non-interactive ARIA roles to interactive HTML elements.
- Make sure label elements have text content and are associated with an input.
- Don't assign interactive ARIA roles to non-interactive HTML elements.
- Don't assign `tabIndex` to non-interactive HTML elements.
- Don't use positive integers for `tabIndex` property.
- Don't include "image", "picture", or "photo" in img alt prop.
- Don't use explicit role property that's the same as the implicit/default role.
- Make static elements with click handlers use a valid role attribute.
- Always include a `title` element for SVG elements.
- Give all elements requiring alt text meaningful information for screen readers.
- Make sure anchors have content that's accessible to screen readers.
- Assign `tabIndex` to non-interactive HTML elements with `aria-activedescendant`.
- Include all required ARIA attributes for elements with ARIA roles.
- Make sure ARIA properties are valid for the element's supported roles.
- Always include a `type` attribute for button elements.
- Make elements with interactive roles and handlers focusable.
- Give heading elements content that's accessible to screen readers (not hidden with `aria-hidden`).
- Always include a `lang` attribute on the html element.
- Always include a `title` attribute for iframe elements.
- Accompany `onClick` with at least one of: `onKeyUp`, `onKeyDown`, or `onKeyPress`.
- Accompany `onMouseOver`/`onMouseOut` with `onFocus`/`onBlur`.
- Include caption tracks for audio and video elements.
- Use semantic elements instead of role attributes in JSX.
- Make sure all anchors are valid and navigable.
- Ensure all ARIA properties (`aria-*`) are valid.
- Use valid, non-abstract ARIA roles for elements with ARIA roles.
- Use valid ARIA state and property values.
- Use valid values for the `autocomplete` attribute on input elements.
- Use correct ISO language/country codes for the `lang` attribute.

### Code Complexity and Quality
- Don't use consecutive spaces in regular expression literals.
- Don't use the `arguments` object.
- Don't use primitive type aliases or misleading types.
- Don't use the comma operator.
- Don't use empty type parameters in type aliases and interfaces.
- Don't write functions that exceed a given Cognitive Complexity score.
- Don't nest describe() blocks too deeply in test files.
- Don't use unnecessary boolean casts.
- Don't use unnecessary callbacks with flatMap.
- Use for...of statements instead of Array.forEach.
- Don't create classes that only have static members (like a static namespace).
- Don't use this and super in static contexts.
- Don't use unnecessary catch clauses.
- Don't use unnecessary constructors.
- Don't use unnecessary continue statements.
- Don't export empty modules that don't change anything.
- Don't use unnecessary escape sequences in regular expression literals.
- Don't use unnecessary fragments.
- Don't use unnecessary labels.
- Don't use unnecessary nested block statements.
- Don't rename imports, exports, and destructured assignments to the same name.
- Don't use unnecessary string or template literal concatenation.
- Don't use String.raw in template literals when there are no escape sequences.
- Don't use useless case statements in switch statements.
- Don't use ternary operators when simpler alternatives exist.
- Don't use useless `this` aliasing.
- Don't use any or unknown as type constraints.
- Don't initialize variables to undefined.
- Don't use the void operators (they're not familiar).
- Use arrow functions instead of function expressions.
- Use Date.now() to get milliseconds since the Unix Epoch.
- Use .flatMap() instead of map().flat() when possible.
- Use literal property access instead of computed property access.
- Don't use parseInt() or Number.parseInt() when binary, octal, or hexadecimal literals work.
- Use concise optional chaining instead of chained logical expressions.
- Use regular expression literals instead of the RegExp constructor when possible.
- Don't use number literal object member names that aren't base 10 or use underscore separators.
- Remove redundant terms from logical expressions.
- Use while loops instead of for loops when you don't need initializer and update expressions.
- Don't pass children as props.
- Don't reassign const variables.
- Don't use constant expressions in conditions.
- Don't use `Math.min` and `Math.max` to clamp values when the result is constant.
- Don't return a value from a constructor.
- Don't use empty character classes in regular expression literals.
- Don't use empty destructuring patterns.
- Don't call global object properties as functions.
- Don't declare functions and vars that are accessible outside their block.
- Make sure builtins are correctly instantiated.
- Don't use super() incorrectly inside classes. Also check that super() is called in classes that extend other constructors.
- Don't use variables and function parameters before they're declared.
- Don't use 8 and 9 escape sequences in string literals.
- Don't use literal numbers that lose precision.

### React and JSX Best Practices
- Don't use the return value of React.render.
- Make sure all dependencies are correctly specified in React hooks.
- Make sure all React hooks are called from the top level of component functions.
- Don't forget key props in iterators and collection literals.
- Don't destructure props inside JSX components in Solid projects.
- Don't define React components inside other components.
- Don't use event handlers on non-interactive elements.
- Don't assign to React component props.
- Don't use both `children` and `dangerouslySetInnerHTML` props on the same element.
- Don't use dangerous JSX props.
- Don't use Array index in keys.
- Don't insert comments as text nodes.
- Don't assign JSX properties multiple times.
- Don't add extra closing tags for components without children.
- Use `<>...</>` instead of `<Fragment>...</Fragment>`.
- Watch out for possible "wrong" semicolons inside JSX elements.

### Correctness and Safety
- Don't assign a value to itself.
- Don't return a value from a setter.
- Don't compare expressions that modify string case with non-compliant values.
- Don't use lexical declarations in switch clauses.
- Don't use variables that haven't been declared in the document.
- Don't write unreachable code.
- Make sure super() is called exactly once on every code path in a class constructor before this is accessed if the class has a superclass.
- Don't use control flow statements in finally blocks.
- Don't use optional chaining where undefined values aren't allowed.
- Don't have unused function parameters.
- Don't have unused imports.
- Don't have unused labels.
- Don't have unused private class members.
- Don't have unused variables.
- Make sure void (self-closing) elements don't have children.
- Don't return a value from a function with the return type 'void'
- Use isNaN() when checking for NaN.
- Make sure "for" loop update clauses move the counter in the right direction.
- Make sure typeof expressions are compared to valid values.
- Make sure generator functions contain yield.
- Don't use await inside loops.
- Don't use bitwise operators.
- Don't use expressions where the operation doesn't change the value.
- Make sure Promise-like statements are handled appropriately.
- Don't use __dirname and __filename in the global scope.
- Prevent import cycles.
- Don't use configured elements.
- Don't hardcode sensitive data like API keys and tokens.
- Don't let variable declarations shadow variables from outer scopes.
- Don't use the TypeScript directive @ts-ignore.
- Prevent duplicate polyfills from Polyfill.io.
- Don't use useless backreferences in regular expressions that always match empty strings.
- Don't use unnecessary escapes in string literals.
- Don't use useless undefined.
- Make sure getters and setters for the same property are next to each other in class and object definitions.
- Make sure object literals are declared consistently (defaults to explicit definitions).
- Use static Response methods instead of new Response() constructor when possible.
- Make sure switch-case statements are exhaustive.
- Make sure the `preconnect` attribute is used when using Google Fonts.
- Use `Array#{indexOf,lastIndexOf}()` instead of `Array#{findIndex,findLastIndex}()` when looking for the index of an item.
- Make sure iterable callbacks return consistent values.
- Use `with { type: "json" }` for JSON module imports.
- Use numeric separators in numeric literals.
- Use object spread instead of `Object.assign()` when constructing new objects.
- Always use the radix argument when using `parseInt()`.
- Make sure JSDoc comment lines start with a single asterisk, except for the first one.
- Include a description parameter for `Symbol()`.
- Don't use spread (`...`) syntax on accumulators.
- Don't use the `delete` operator.
- Don't access namespace imports dynamically.
- Don't use namespace imports.
- Declare regex literals at the top level.
- Don't use `target="_blank"` without `rel="noopener"`.

### TypeScript Best Practices

---
> Source: [sunshineLixun/ShipFullStack](https://github.com/sunshineLixun/ShipFullStack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
