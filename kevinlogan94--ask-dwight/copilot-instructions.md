## ask-dwight

> This document outlines the standards for creating Vue.js components. It builds upon the established TypeScript standard and is designed to ensure all components are robust, consistent, and maintainable.


This document outlines the standards for creating Vue.js components. It builds upon the established TypeScript standard and is designed to ensure all components are robust, consistent, and maintainable.

1. Component Structure

<script setup> is Mandatory: All new components must use the <script setup> syntax. This is the most efficient and straightforward approach for Vue 3.

Block Order: The blocks within a .vue file must be in the following order: <template>, then <script>, then <style>.

Scoped Styles: Component styles must be scoped using the <style scoped> attribute to prevent CSS conflicts.

```
<template>
  </template>

<script setup lang="ts">
  // TypeScript logic here
</script>

<style scoped>
  /* Component-specific CSS here */
</style>
```


2. Props & Events

Strict Prop Validation: All props must be defined using the object syntax with, at a minimum, type and required properties. This ensures every prop is explicitly documented. Default values should be provided via the default property.

```
import type { PropType } from 'vue';
import type { User } from '@/types/user.types';

defineProps({
  user: {
    type: Object as PropType<User>,
    required: true,
  },
  isSaving: {
    type: Boolean,
    required: false, // Explicitly not required
  },
  theme: {
    type: String as PropType<'dark' | 'light'>,
    default: 'light',
  },
});
```


Prop Casing: Props must be declared in camelCase in the script and consumed in kebab-case in the template.

Emits Definition: All events a component can emit must be defined using the defineEmits macro.

3. Reactivity & Template Syntax

Reactivity with ref: For consistency, all reactive state must be created using ref. This applies to both primitive types (strings, numbers) and objects.

```
import { ref } from 'vue';

const count = ref(0);
const user = ref({ id: 1, name: 'Jane Doe' });

function increment() {
  // Always access/mutate via .value
  count.value++;
  user.value.name = 'John Doe';
}
```


Use computed for Derived State: Any data that is derived from other reactive state must be a computed property.

Directive Shorthands: Always use the shorthand syntax for directives: : for v-bind, @ for v-on, and # for v-slot.

v-for Keys: Every v-for loop must have a unique :key attribute bound to a stable ID. Do not use the loop's index as a key.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevinlogan94) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
