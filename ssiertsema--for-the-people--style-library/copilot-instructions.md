## for-the-people-style-library

> | Token type | Ratio | Reasoning |

# Style Library Development Guidelines

## Design Rationale

### Scale ratios
| Token type | Ratio | Reasoning |
|------------|-------|-----------|
| **Spacers** | Golden ratio (φ ≈ 1.618) | Fewer steps cover the full range; naturally harmonious progression |
| **Radii** | Fibonacci/Golden ratio | Same benefits as spacing; 6 values sufficient for all rounding needs |
| **Shadows** | Doubling (2×) | Larger jumps needed for perceptible depth; matches physical light falloff |
| **Typography** | Major third (~1.25) | Finer control needed for readable text; golden ratio jumps too aggressively |

### Naming convention
All scales use **t-shirt sizing** (3xs, 2xs, xs, s, m, l, xl, 2xl, etc.) for consistency across the design system. This makes it easier for developers and AI agents to reason about relative sizes without memorizing pixel values.

### Pill shapes
For fully rounded elements (pills, circles), use `border-radius: 9999px` directly. This is intentionally not a variable — it's a fixed styling pattern, not a scale value.

## Styling Rules

All components must use variables and mixins from the style library layer. No magic values allowed.

### Required imports in component SCSS files:
```scss
@use "@pzh-temporary/style-library/src/scss/variables/colors.scss" as *;
@use "@pzh-temporary/style-library/src/scss/variables/spacers.scss" as *;
@use "@pzh-temporary/style-library/src/scss/variables/radii.scss" as *;
@use "@pzh-temporary/style-library/src/scss/mixins/typography.scss" as *;
@use "@pzh-temporary/style-library/src/scss/mixins/levitation.scss" as *;
```

### Spacer scale (golden ratio):
| Variable | Value |
|----------|-------|
| `$space-3xs` | 2px |
| `$space-2xs` | 4px |
| `$space-xs` | 6px |
| `$space-s` | 10px |
| `$space-m` | 16px |
| `$space-l` | 26px |
| `$space-xl` | 42px |
| `$space-2xl` | 68px |
| `$space-3xl` | 110px |
| `$space-4xl` | 178px |

### Radii scale (Fibonacci/golden ratio):
| Variable | Value |
|----------|-------|
| `$radius-xs` | 2px |
| `$radius-s` | 3px |
| `$radius-m` | 5px |
| `$radius-l` | 8px |
| `$radius-xl` | 13px |
| `$radius-2xl` | 21px |

Note: For pill shapes, use `border-radius: 9999px` directly (not a variable).

### Shadow scale (layered, 2× progression):
| Variable | Y-offset | Blur | Use case |
|----------|----------|------|----------|
| `$shadow-xs` | 1px | 2px | Hover states, subtle depth |
| `$shadow-s` | 2px | 4px | Cards, buttons |
| `$shadow-m` | 4px | 8px | Dropdowns, popovers |
| `$shadow-l` | 8px | 16px | Modals, dialogs |
| `$shadow-xl` | 16px | 32px | Toasts, high elevation |

Each shadow combines a key shadow (directional) and ambient shadow (diffuse) for realistic depth. Use via mixin: `@include levitate("m");`

### Examples:
```scss
// Correct
padding: $space-m;
color: $color-blue-900;
border-radius: $radius-m;
@include typography("text", "s");
@include levitate("s");

// Wrong - no magic values
padding: 16px;
color: #1a2b3c;
border-radius: 4px;
font-size: 14px;
box-shadow: 0 2px 4px rgba(0,0,0,0.1);
```

## Component Replication (PrimeVue)

When replicating a PrimeVue component, follow these steps:

### 1. Research the PrimeVue component
- Fetch the component API from PrimeVue GitHub: `https://raw.githubusercontent.com/primefaces/primevue/master/packages/primevue/src/{component}/`
- Note all props, events, slots, and features
- Adapt prop names to library conventions (e.g., `isDisabled` instead of `disabled`)

### 2. Create component token file
Create token file: `tokens/components/{component-name}.json`

Use semantic token references (not primitive tokens directly):
- Spacing: `{inset.s}`, `{inset.m}`, `{inline.s}`, `{stack.m}`
- Radius: `{radius.subtle}`, `{radius.default}`, `{radius.rounded}`
- Colors: `{neutral.400}`, `{brand.primary}`, `{text.default}`, `{surface.canvas}`
- Shadows: `{shadow.s}`, `{shadow.m}`

```json
{
  "componentName": {
    "size": {
      "sm": { "value": "32px", "type": "dimension" },
      "md": { "value": "40px", "type": "dimension" }
    },
    "background": {
      "default": { "value": "{surface.canvas}", "type": "color" },
      "hover": { "value": "{neutral.50}", "type": "color" }
    },
    "border": {
      "color": {
        "default": { "value": "{neutral.300}", "type": "color" }
      }
    }
  }
}
```

### 3. Create component files
Create folder: `src/components/{ComponentName}/`

**{ComponentName}.vue:**
```vue
<template>
  <div class="{component-name}" :class="additionalClasses">
    <!-- component markup -->
  </div>
</template>

<style src="./{ComponentName}.scss"></style>

<script setup>
import { computed } from "vue";

const props = defineProps({
  // props with validators
});

const additionalClasses = computed(() =>
  [`{component-name}--size-${props.size}`].join(" ")
);
</script>
```

**{ComponentName}.scss:**
```scss
@use "../../../dist/scss/tokens" as *;

$c: {component-name};

.#{$c} {
  // Use CSS custom properties with SCSS fallbacks
  background-color: var(--componentName-background-default, $componentName-background-default);
  border: 1px solid var(--componentName-border-color-default, $componentName-border-color-default);
}

.#{$c}--size-sm {
  height: var(--componentName-size-sm, $componentName-size-sm);
}
```

### 4. Build tokens
Run `npx tsx scripts/build-tokens.ts` to generate CSS/SCSS output.

### 5. Add to documentation
Create documentation page: `server/pages/components/form/{component-name}.vue`

Use aliases for clean imports:
- `@components` → `src/components`
- `@lib` → `src`
- `@tokens` → `dist`

```vue
<script setup lang="ts">
import ComponentName from '@components/ComponentName/ComponentName.vue'
import '@components/ComponentName/ComponentName.scss'

const componentProps = [
  { name: 'propName', type: 'String', default: '"value"', description: 'Description' }
]
</script>

<template>
  <div class="page">
    <PageHeader title="ComponentName" description="Component description." />
    <ComponentDemo name="ComponentName" :props="componentProps">
      <ComponentName prop="value" />
    </ComponentDemo>
  </div>
</template>
```

### 6. Add to navigation
Update `server/layouts/default.vue` to add the component to the sidebar navigation.

### 7. Update checklist
Mark component as complete in `target-components.md`

## Documentation Server

The documentation server is a Nuxt 3 app in `/server`.

### Path Aliases (configured in nuxt.config.ts)
| Alias | Path |
|-------|------|
| `@lib` | `../src` |
| `@components` | `../src/components` |
| `@tokens` | `../dist` |

### Running the server
```bash
cd server && npm run dev
```

### Structure
- `server/pages/components/form/` - Form component documentation
- `server/pages/style-library/` - Design token documentation
- `server/layouts/default.vue` - Navigation sidebar

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SSiertsema)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SSiertsema)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
