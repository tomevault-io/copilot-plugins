## create-new-vue-motion-icon

> Here's a step-by-step guide for converting React animated Lucide icons to Vue components in this project:

# Converting React Lucide Icons to Vue: Guide

Here's a step-by-step guide for converting React animated Lucide icons to Vue components in this project:

## 1. Basic Structure

```vue
<script setup lang="ts">
import { motion } from 'motion-v';

interface Props {
  size?: number;
  class?: string;
}

const props = withDefaults(defineProps<Props>(), {
  size: 28,
  class: '',
});

const emit = defineEmits<{
  startAnimation: [];
  stopAnimation: [];
}>();

// Define animation variants here

const isControlled = ref(false);
const currentState = ref('normal');

const startAnimation = () => {
  currentState.value = 'animate';
};

const stopAnimation = () => {
  currentState.value = 'normal';
};

const handleMouseEnter = () => {
  if (!isControlled.value) {
    startAnimation();
  } else {
    emit('startAnimation');
  }
};

const handleMouseLeave = () => {
  if (!isControlled.value) {
    stopAnimation();
  } else {
    emit('stopAnimation');
  }
};

defineExpose({
  startAnimation,
  stopAnimation,
});
</script>

<template>
  <div
    :class="[
      'cursor-pointer select-none p-2 hover:bg-accent rounded-md transition-colors duration-200 flex items-center justify-center',
      props.class,
    ]"
    @mouseenter="handleMouseEnter"
    @mouseleave="handleMouseLeave"
  >
    <svg>
      <!-- SVG elements here -->
    </svg>
  </div>
</template>
```

## 2. Animation Pattern Conversion

When converting React motion animations to Vue:

| React | Vue |
|-------|-----|
| `animate={controls}` | `:animate="currentState"` |
| `variants={...}` | `:variants="iconVariants"` |
| `<motion.svg>` | `<svg>` with `<motion.path>` inside |
| `transition={{ ... }}` | Include in each variant or as separate prop |

## 3. Converting Animation Arrays

- Keep animation arrays in the same format: `[val1, val2, val3, ...]`
- Include all keyframe points from React
- Use `duration` and `bounce` parameters for spring animations
- For complex keyframes, use `transition` inside the variant

Example:
```typescript
const searchVariants = {
  normal: { 
    x: 0, 
    y: 0,
    transition: {
      duration: 0.3,
      type: 'spring',
      stiffness: 400,
      damping: 25
    }
  },
  animate: { 
    x: [0, 0, -3, 0],  // Same array as React
    y: [0, -4, 0, 0],  // Same array as React
    transition: {
      duration: 1,
      bounce: 0.3      // Same as React
    }
  }
};
```

## 4. SVG Path Conversion

- Use `motion.path`, `motion.circle`, etc. for elements that need animation
- Keep all SVG attributes the same (viewBox, fill, stroke, etc.)
- Add `style="overflow: visible"` to the SVG element if needed

## 5. Adding to index.vue

After creating a new icon component in `app/components/Lucide/`, add it to `app/pages/index.vue`:

```vue
<AnimatedIconButton>
  <template #default="{ ref, controlled }">
    <LucideYourNewIcon
      :ref="ref"
      :controlled="controlled"
      :class="primaryColor"
    />
  </template>
</AnimatedIconButton>
```

## Common Gotchas

1. Don't use arrays with spring animations for x/y - use only two keyframes or keyframes with tween animations
2. Remember all Vue components in the Lucide directory are auto-imported with the "Lucide" prefix
3. Ignore TypeScript errors about `motion-v` - they appear in all components
4. Keep animations simple - one normal state and one animate state
5. Don't add unnecessary timers or complex logic - just toggle the state

---
> Source: [fayazara/animated-lucide-vue](https://github.com/fayazara/animated-lucide-vue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
