## design-sytem-svelte-components

> Convert Figma-to-code Svelte output into a proper design-system component


# Figma-to-Component Conversion

When the user pastes Figma-generated Svelte code, follow this pipeline exactly.
Full details and examples are in `docs/definitions/figma-to-component.md`.

## Status check — do this FIRST

Before doing anything else, read the relevant `docs/definitions/<component>.md`.

- If the file has `**Status:** \`done\`` → **stop immediately**. Component exists, do not regenerate.
- If status is `pending` or missing → continue with the pipeline below.
- After completing step 5, update the status line in the doc to `done` and add the component path.

Status line format:
```md
> **Status:** `done` — Component lives at `src/lib/components/<name>/`. Do not regenerate.
```

---

## Pipeline (do in order)

### 1 — Identify variants and states
Read the pasted code and list every visual state (size, color, disabled, etc.).
Each state becomes a key in a `tv()` variant block — not a separate component.

### 2 — Map values to design-system tokens
Replace every arbitrary CSS value with the nearest design-system token from `tailwind-presets/v4/theme.css`:

| Figma output | Replace with |
|---|---|
| `bg-[#0032A0]` | `bg-primary` |
| `text-[#FFFFFF]` | `text-common-white` |
| `w-[100px]` `h-[100px]` | `size-*` token or `w-*`/`h-*` from spacing scale |
| `font-['Gilroy']` | `font-heading` |
| `font-['Suisse_Int\'l']` | `font-body` |
| `rounded-[999px]` | `rounded-full` |
| Absolute positioning (`top-`, `left-`, `position-absolute`) | Remove — use flex/grid layout |

### 3 — Factor with tailwind-variants
Create one `tv()` call per logical component. Multiple visual dimensions = multiple variant keys.

```ts
// ❌ BAD — Figma layers as separate CSS classes
'.button-primary { background: #0032A0; ... }'
'.button-secondary { background: transparent; ... }'

// ✅ GOOD — factored with tv()
const buttonVariants = tv({
  base: 'font-body font-medium rounded-full flex items-center',
  variants: {
    variant: {
      primary: 'bg-primary text-common-white',
      secondary: 'bg-transparent text-primary border border-primary'
    },
    size: { default: 'px-4 py-3', slim: 'px-3 py-2 text-d3' }
  }
});
```

### 4 — Write the Svelte 5 component
Follow the existing component pattern:
- `$props()` with explicit types — no `$$props` spread
- `bits-ui` primitive as root when semantics demand it (button, checkbox, etc.)
- `cn(variantFn({ ...props }), className)` for the class binding
- `{children}` snippet (not `{child}`) for slot content

### 5 — Add the Storybook story and test
After generating the component, always produce:
- `src/stories/ComponentNameStory.svelte` + `src/stories/component-name.stories.ts`
- `src/lib/components/<name>/<name>.test.ts`

See `component-standards.mdc` for the exact requirements.

---
> Source: [copa-airlines-digital-marketing/design-sytem-svelte-components](https://github.com/copa-airlines-digital-marketing/design-sytem-svelte-components) — distributed by [TomeVault](https://tomevault.io/claim/copa-airlines-digital-marketing).
<!-- tomevault:4.0:copilot_instructions:2026-04-18 -->
