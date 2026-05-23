## code-spacing

> Enforces consistent spacing throughout the codebase

# Rules

## Use consistent spacing
The following patterns are required:
- Empty line between code blocks
- Maximum 1 consecutive empty line
- Empty line after imports
- Empty line before/after functions
- Empty line before exports
- Empty line after class members
- Empty line before return statements

# Examples

## Valid
```typescript
import { ref, watch } from 'vue'
import { TextInputBare } from 'vue-bare'

const props = defineProps<Props>()
const emit = defineEmits(['update:modelValue'])

const handleInput = (event: Event) => {
  const input = event.target as HTMLInputElement
  const value = input.value

  return value
}

export const exportedFunction = (param: string) => {
  return param.toUpperCase()
}
```

## Invalid
```typescript
import { ref, watch } from 'vue'
import { TextInputBare } from 'vue-bare'
const props = defineProps<Props>()
const emit = defineEmits(['update:modelValue'])
const handleInput = (event: Event) => {
  const input = event.target as HTMLInputElement
  const value = input.value
  return value
}
export const exportedFunction = (param: string) => {
  return param.toUpperCase()
}
```

---
> Source: [ericvera/whatsapp-cloudapi](https://github.com/ericvera/whatsapp-cloudapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
