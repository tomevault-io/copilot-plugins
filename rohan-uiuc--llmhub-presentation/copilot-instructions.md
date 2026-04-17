## llmhub-presentation

> Vue Components & Custom Component Development

# Vue Components & Custom Component Development

## Creating Custom Components
- Place Vue components in `components/` directory
- Components are automatically available in all slides
- Example: [Counter.vue](mdc:components/Counter.vue) demonstrates reactive counter
- Use Composition API with `<script setup lang="ts">`

## Component Usage in Slides
```vue
<Counter :count="10" />
```

## Vue 3 Composition API Patterns
```vue
<script setup lang="ts">
import { ref } from 'vue'

const props = defineProps({
  count: {
    default: 0,
  },
})

const counter = ref(props.count)
</script>
```

## Styling Components
- Use UnoCSS/Tailwind classes for utility styling
- Scoped styles with `<style>` tags in components
- Example from [Counter.vue](mdc:components/Counter.vue): `flex="~"`, `w="min"`, `border="~ main rounded-md"`
- Hover effects: `hover:bg="gray-400 opacity-20"`

## Component Best Practices
1. Use TypeScript with `<script setup lang="ts">`
2. Define props with proper types and defaults
3. Use reactive refs for component state
4. Apply UnoCSS classes for consistent styling
5. Test components in development mode with `bun run dev`
6. Keep components focused and reusable

## Interactive Features in Components
- Click handlers: `@click="counter += 1"`
- Reactive data with `ref()` and `computed()`
- Props passing from slides to components
- Event handling and component communication

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rohan-uiuc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
