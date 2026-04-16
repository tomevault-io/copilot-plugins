## scrapyard-sv-frontend

> You are an expert Svelte developer. You are given a task to write Svelte 5 code. You should be familiar with Svelte 4. However, Svelte 5 is a major new version with some breaking changes.

You are an expert Svelte developer. You are given a task to write Svelte 5 code. You should be familiar with Svelte 4. However, Svelte 5 is a major new version with some breaking changes.

## Core Changes in Svelte 5

Svelte 5 introduces a new reactivity system based on **runes**. These are special symbols prefixed with `$` that instruct the Svelte compiler. Runes fundamentally change how reactivity and component APIs are defined.

**Key Concepts that are DIFFERENT in Svelte 5 for You to Remember:**

- **Runes-based Reactivity:** Svelte 5 uses runes for explicit reactivity, replacing implicit reactivity of Svelte 4. **You must use runes to create reactive code.**
- **Event Attributes:** Event handling is now done using standard HTML event attributes (e.g., `onclick`) instead of `on:` directives. **Use standard event attributes in your Svelte 5 code.**
- **Snippets instead of Slots:** Slots are replaced by snippets, a more flexible and powerful way to handle content projection. **Generate snippets instead of slots for content projection.**
- **Component API Changes:** Components are no longer classes; instantiation and interaction methods have changed. **Adjust your component instantiation and interaction code to use the new Svelte 5 API.**
- **`.svelte.js` and `.svelte.ts` files:** Introduction of these files for reusable reactive logic and shared state outside components. **You can now generate reusable reactive logic in these files.**

## Template Syntax Changes

### 1. Reactivity: `$state`, `$derived`, `$effect` (NEW RUNES - YOU MUST USE THESE)

- **`$state(value)`:** Replaces `let` for declaring reactive variables. Variables must be explicitly made reactive using `$state()`.

  ```diff
  - let count = 0; // Svelte 4 - implicitly reactive
  + let count = $state(0); // Svelte 5 - explicitly reactive using $state
  ```

  **When you need to make a variable reactive, use `$state(value)` instead of `let`.**

- **`$derived(expression)` / `$derived.by(() => { ... })`:** Replaces `$:` for derived values. Explicitly declares derived state.

  ```diff
  - $: doubled = count * 2; // Svelte 4 - reactive statement
  + const doubled = $derived(count * 2); // Svelte 5 - derived state using $derived
  ```

  **For derived values, use `$derived(expression)` or `$derived.by(() => { ... })` instead of `$:`.**

- **`$effect(() => { ... })` / `$effect.pre(() => { ... })`:** Replaces `$:` for side effects. Explicitly declares side effects. `$effect.pre` runs before DOM updates.
  ```diff
  - $: console.log(count); // Svelte 4 - reactive statement for side effect
  + $effect(() => { console.log(count); }); // Svelte 5 - side effect using $effect
  ```
  **For side effects, use `$effect(() => { ... })` or `$effect.pre(() => { ... })` instead of `$:`. Remember `$effect.pre` runs before DOM updates.**

### 2. Props: `$props` (NEW RUNE - YOU MUST USE THIS)

- **`let { prop1, prop2, ... } = $props()`:** Replaces `export let prop1; export let prop2;` for declaring component props. Destructuring is used.
  ```diff
  - export let message: string; // Svelte 4 - export let for props
  + let { message }: { message: string } = $props(); // Svelte 5 - $props for props
  ```
  **To declare component props, use `let { prop1, prop2, ... } = $props();` instead of `export let`.** Fallback values are still supported in destructuring: `let { prop = 'default' } = $props();`
- You must always give types to props. This is done by having a `type Props = { ... }` declaration, and then you should do `const { prop1, prop2, ... }: Props = $props();`. Note that the `$props` rune is not generic; DO NOT DO `$props<Props>()`.

### 3. Events: Event Attributes (CHANGED - ADJUST YOUR EVENT HANDLING)

- **`onclick={handler}`:** Replaces `on:click={handler}` for DOM events. Event handlers are now standard HTML attributes.

  ```diff
  - <button on:click={handleClick}>Click me</button> // Svelte 4 - on:directive
  + <button onclick={handleClick}>Click me</button>     // Svelte 5 - onclick attribute
  ```

  **When adding event handlers to DOM elements, use standard HTML event attributes like `onclick`, `oninput`, etc., instead of `on:click`, `on:input`, etc.** Event modifiers (`|preventDefault`, `|stopPropagation`, etc.) are **deprecated**. **Generate `event.preventDefault()` etc. inside the handler instead.** `capture`, `passive`, `nonpassive` modifiers are replaced with event attribute variants like `onclickcapture`.

- **Component Events:** `createEventDispatcher` is **deprecated**. Use callback props instead.

  ```diff
  // Svelte 4 - Component Events
  - import { createEventDispatcher } from 'svelte';
  - const dispatch = createEventDispatcher();
  - dispatch('myEvent', payload);
  - <Component on:myEvent={handler} />

  // Svelte 5 - Callback Props
  + let { onMyEvent } = $props();
  + onMyEvent(payload);
  + <Component onMyEvent={handler} />
  ```

  **For components to communicate events to their parents, use callback props instead of `createEventDispatcher` and `on:myEvent`. Generate props like `onMyEvent` and call them directly.**

### 4. Slots & Snippets: Snippets (REPLACED - GENERATE SNIPPETS INSTEAD OF SLOTS)

- **`{#snippet name(params)}...{@render name(args)}`:** Replaces `<slot>` for content projection. Snippets are more flexible and powerful.

  ```diff
  // Svelte 4 - Slots
  - <Component>
  -   <slot />
  - </Component>

  // Svelte 5 - Snippets
  + <Component>
  +   {#snippet default()} Content {/snippet}
  + </Component>

  // Component.svelte
  - <slot>Fallback Content</slot> // Svelte 4
  + {@render children?.()} // Svelte 5 for default content
  ```

  **When you want to project content into components, generate snippets using `{#snippet name(params)}...{/snippet}` and render them using `{@render name(args)}`. Replace `<slot>` elements with `{@render children?.()}` for default content.** Named slots (`<slot name="...">`) are replaced by named snippets passed as props:

  ```diff
  // Svelte 4 - Named Slots
  - <Component>
  -   <div slot="header">Header Content</div>
  - </Component>
  - <slot name="header"></slot>

  // Svelte 5 - Named Snippets
  + <Component>
  +   {#snippet header()}Header Content{/snippet}
  + </Component>
  + {@render header?.()} // Render header snippet
  ```

### 5. `bind:` Directive (MINOR CHANGES - CONTINUE USING `bind:`)

- `bind:value`, `bind:checked`, `bind:group`, `bind:files`, `bind:open` and dimension bindings (`bind:clientWidth`, etc.) largely remain the same but now leverage the new reactivity system. **Continue to use `bind:` as before for these bindings.**
- `bind:this` is still used for DOM node and component instance references. **Continue to use `bind:this` to get DOM node and component references.**
- **`bind:property={get, set}`:** Function bindings are **new** in Svelte 5.9.0+, allowing custom getter/setter functions for bindings, enabling validation and transformations. **You can now use function bindings like `bind:value={get, set}` for advanced binding scenarios.**

### 6. Control Flow Blocks (`{#if}`, `{#each}`, `{#await}`, `{#key}`) (LARGELY UNCHANGED - CONTINUE TO GENERATE THESE BLOCKS)

- `{#if}`, `{:else if}`, `{:else}`, `{#each}`, `{:else}`, `{#await}`, `{:then}`, `{:catch}`, `{#key}` blocks remain syntactically similar. **Continue to generate these control flow blocks as before.**
- `{#each ... (key)}` is now crucial for animations and keyed updates when reordering lists. **When generating `{#each}` blocks for lists that might reorder, ensure you include a `(key)` expression for efficient updates and animations.**
- `{#const area = ...}` is a **new** tag for declaring local constants within blocks. **You can use `{#const}` to declare local constants inside template blocks.**

### 7. Special Elements (`<svelte:window>`, `<svelte:document>`, `<svelte:head>`, `<svelte:body>`, `<svelte:element>`, `<svelte:options>`, `<svelte:boundary>`) (LARGELY UNCHANGED - CONTINUE TO GENERATE THESE ELEMENTS)

- Special elements mostly function the same, but are integrated with the new reactivity system. **Continue to generate these special elements as needed.**
- `<svelte:boundary>` is a **new** element introduced in Svelte 5.3.0 for error boundaries. **You can use `<svelte:boundary>` to create error boundaries in your components.**
- `<svelte:options runes={true/false}>` is used to explicitly control runes mode. **Use `<svelte:options runes={true}>` to force a component into runes mode if needed.**
- `<svelte:element this={expression}>` is used for dynamic element rendering. **Continue to use `<svelte:element>` for rendering dynamic elements.**

### 8. `use:` Directive (LARGELY UNCHANGED - CONTINUE TO GENERATE `use:`)

- Actions using `use:` directive work similarly but now primarily utilize `$effect` within action functions for setup and teardown, replacing the older `update`/`destroy` return object pattern. **Continue to use `use:` directives for actions, but ensure action functions use `$effect` for reactive behavior.**

### 9. Transitions & Animations (`transition:`, `in:`, `out:`, `animate:`) (LARGELY UNCHANGED - CONTINUE TO GENERATE TRANSITIONS & ANIMATIONS)

- Transitions and Animations are generally the same but benefit from the new reactivity system. **Continue to generate transitions and animations as before.**
- Transitions are **local by default** in Svelte 5. Use `|global` modifier for global transitions. **Be aware that transitions are now local by default. If global transitions are needed, generate the `|global` modifier.**

### 10. `@html`, `@debug`, `@const`, `@render` (NEW TAGS - LEARN TO GENERATE THESE TAGS)

- `{@html ...}`: Remains the same for injecting raw HTML. **Continue to generate `{@html ...}` for raw HTML injection.**
- `{@debug ...}`: Remains the same for debugging. **Continue to generate `{@debug ...}` for debugging purposes.**
- `{@const ...}`: **New** tag for declaring local constants in template blocks. **You can generate `{@const ...}` to declare local constants within template blocks.**
- `{@render ...}`: **New** tag for rendering snippets. **Use `{@render ...}` tags to render snippets instead of `<slot>` elements.**

## Runtime API Changes

- **Component Instantiation:** `new Component(options)` is **deprecated**. Use `mount(Component, options)` and `hydrate(Component, options)` instead for imperative component creation. **When programmatically creating components, use `mount(Component, options)` or `hydrate(Component, options)` instead of `new Component(options)`.**
- **Component API:** `$set`, `$on`, `$destroy` methods on component instances are **deprecated**. Use `$state` for props, callback props for events, and `unmount()` for destruction. **Avoid generating `$set`, `$on`, and `$destroy` calls on component instances. Use runes for reactivity and `unmount()` for component removal.**
- **Server-Side Rendering:** `Component.render()` is **deprecated**. Use `render(Component, options)` from `svelte/server`. **For server-side rendering, use `render(Component, options)` instead of `Component.render()`.**

## New Concepts

- **.svelte.js and .svelte.ts files:** These files allow using runes outside of `.svelte` components, enabling reusable reactive logic and shared state. **You can generate `.svelte.js` and `.svelte.ts` files for reusable reactive logic and shared state.**
- **Runes:** `$state`, `$derived`, `$effect`, `$props`, `$bindable`, `$inspect`, `$host` are the core of Svelte 5 reactivity. **Focus on using runes as the primary mechanism for reactivity in Svelte 5 code.**

## Deprecated/Removed Features

- **Implicit Reactivity (`let` at top level):** Replaced by `$state`. **Do not rely on implicit reactivity. Always use `$state` for reactive variables.**
- **Reactive Statements (`$:`):** Replaced by `$derived` and `$effect`. **Do not use `$:`. Use `$derived` and `$effect` instead.**
- **`export let` for props:** Replaced by `$props`. **Do not use `export let` for props. Use `$props` instead.**
- **`createEventDispatcher` and `on:` directive for component events:** Replaced by callback props. **Do not use `createEventDispatcher` or `on:` directive for component events. Use callback props instead.**
- **Slots (`<slot>`):** Replaced by snippets and render tags. **Do not generate `<slot>` elements for content projection. Use snippets and render tags instead.**
- **`beforeUpdate`, `afterUpdate` lifecycle hooks:** Replaced by `$effect.pre` and `$effect`. **Do not use `beforeUpdate` and `afterUpdate`. Use `$effect.pre` and `$effect` instead.**
- **`svelte/store` stores for basic reactivity:** While still usable, runes are the preferred approach for most reactivity needs. **Prioritize runes over `svelte/store` for basic reactivity.**
- **`immutable` and `accessors` options in runes mode:** These compiler options have no effect in runes mode. **Do not generate code relying on `immutable` and `accessors` compiler options in runes mode.**
- **`svelte:component this="..."` for dynamic components:** Components are dynamic by default in Svelte 5. `<svelte:component>` is mostly unnecessary. **Avoid generating `<svelte:component this="...">` unless absolutely necessary for legacy reasons.**

## Important Notes for You, the AI

- **Focus on Runes:** **Your generated Svelte 5 code MUST primarily use runes (`$state`, `$derived`, `$effect`, `$props`, `$bindable`) for reactivity.**
- **Prefer Event Attributes:** **Always use HTML event attributes (e.g., `onclick`) for DOM events, not `on:` directives.**
- **Use Snippets for Content Projection:** **Generate snippets and render tags for component composition instead of `<slot>` elements.**
- **Component Instantiation with `mount`:** **When you need to programmatically create components, use `mount(Component, options)` or `hydrate(Component, options)`.**
- **Modern Browser Target:** **Assume a modern browser environment for Svelte 5 code generation.**
- **Stricter HTML Structure:** **Generate valid HTML as Svelte 5 is stricter about HTML structure.**
- **`.svelte.js`/`.svelte.ts` for Logic:** **Utilize `.svelte.js` and `.svelte.ts` files when you need to generate reusable reactive logic.**

This `.cursorrules` file should guide you in generating code that leverages the new features of Svelte 5 and avoids deprecated Svelte 4 patterns. Pay close attention to the sections marked "YOU MUST..." or "ADJUST YOUR..." as these highlight the most crucial changes affecting your code generation strategy.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ByteAtATime) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
