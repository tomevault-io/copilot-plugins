## tailwindcss4

> Tailwind CSS 4 Information


# TailwindCSS 4

- `tailwind.config.js` must never exist
- Use CSS-first configuration with `@theme` directive instead of JavaScript config `tailwind.config.js`:

```css
@import 'tailwindcss';

@theme {
    --font-display: 'Satoshi', 'sans-serif';
    --breakpoint-3xl: 1920px;
    --color-avocado-500: oklch(0.84 0.18 117.33);
    --ease-fluid: cubic-bezier(0.3, 0, 0, 1);
}
```

## Theme Configuration

- Never use a dynamic utility value. Only use Tailwind's built in values.
- Use CSS variables for all design tokens:

```css
/* In CSS */
.custom-element {
    background-color: var(--color-blue-500);
    font-family: var(--font-sans);
}
```

- Available CSS variable namespaces:

    - `--color-*`: Colors (e.g., `--color-blue-500`)
    - `--font-*`: Font families (e.g., `--font-sans`)
    - `--text-*`: Font sizes (e.g., `--text-xl`)
    - `--font-weight-*`: Font weights (e.g., `--font-weight-bold`)
    - `--spacing-*`: Spacing values (e.g., `--spacing-4`)
    - `--radius-*`: Border radius (e.g., `--radius-md`)
    - `--shadow-*`: Box shadows (e.g., `--shadow-lg`)

- Override entire namespaces or the whole theme:

```css
@theme {
    /* Override all font variables */
    --font-*: initial;

    /* Override the entire theme */
    --*: initial;
}
```

## 3D Transforms

- Example: Use 3D transforms with new utilities:

```html
<!-- Enable 3D transforms -->
<div class="translate-z-4 rotate-x-12 rotate-y-6 perspective-distant transform-3d">3D transformed element</div>

<!-- Control backface visibility -->
<div class="rotate-y-180 backface-hidden transform-3d">Card back (hidden when flipped)</div>
```

## Enhanced Gradients

- Example: Use new gradient syntax and features:

```html
<!-- Linear gradient with specific angle -->
<div class="bg-linear-45 from-blue-500 to-purple-500">45-degree gradient</div>

<!-- Gradient with specific color space interpolation -->
<div class="bg-linear-to-r/oklch from-blue-500 to-red-500">Linear gradient with OKLCH interpolation</div>

<!-- Conic and radial gradients -->
<div class="bg-conic from-red-500 via-yellow-500 to-green-500">Conic gradient</div>

<div class="bg-radial-[at_25%_25%] from-amber-500 to-transparent">Radial gradient with custom position</div>
```

## New Variants

- Example: Use composable variants by chaining them:

```html
<div class="group">
    <!-- Only visible when parent has data-active attribute and is hovered -->
    <span class="opacity-0 group-has-data-active:group-hover:opacity-100"> Conditionally visible </span>
</div>
```

- Example: Use new variants:

```html
<!-- Styles applied during CSS transitions -->
<div class="opacity-0 transition starting:opacity-100">Fade in on initial render</div>

<!-- Target elements that are not in a specific state -->
<div class="not-first:mt-4">Margin top on all but first item</div>

<!-- Target specific nth-child positions -->
<ul>
    <li class="nth-3:bg-gray-100">Every third item has gray background</li>
</ul>

<!-- Target all descendants -->
<div class="**:text-gray-800">All text inside is gray-800</div>
```

## Custom Extensions

- Example: Create custom utilities with `@utility` directive:

```css
@utility tab-4 {
  tab-size: 4;
}

/* Usage */
<pre class="tab-4">
  Indented with tabs
</pre>
```

- Example: Create custom variants with `@variant` directive:

```css
@variant pointer-coarse (@media (pointer: coarse));
@variant theme-midnight (&:where([data-theme="midnight"] *));

/* Usage */
<button class="pointer-coarse:p-4">
  Larger padding on touch devices
</button>
```

- Example: Use plugins with `@plugin` directive:

```css
@plugin "@tailwindcss/typography";
```

## Breaking Changes

- Example: Use new syntax for CSS variables in arbitrary values:

```html
<!-- Old way -->
<div class="bg-[--brand-color]">Using CSS variable</div>

<!-- New way -->
<div class="bg-(--brand-color)">Using CSS variable</div>
```

- Example: Use renamed utilities:

```html
<!-- Old way -->
<div class="rounded-sm shadow-sm blur-sm"></div>

<!-- New way -->
<div class="rounded-xs shadow-xs blur-xs"></div>
```

## Advanced Configuration

- Example: Add a prefix to all Tailwind classes:

```css
@import "tailwindcss" prefix(tw);

/* Results in classes like: */
<div class="tw:flex tw:bg-blue-500 tw:hover:bg-blue-600">
  Prefixed classes
</div>
```

- Example: Configure dark mode:

```css
@import "tailwindcss";
@variant dark (&:where(.dark, .dark *));

/* Usage */
<div class="dark">
  <p class="text-gray-900 dark:text-white">
    Dark mode text
  </p>
</div>
```

- Example: Customize container:

```css
@utility container {
    margin-inline: auto;
    padding-inline: 2rem;
}
```

---
> Source: [appdotbuilder/damkar-morowali-utara](https://github.com/appdotbuilder/damkar-morowali-utara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
