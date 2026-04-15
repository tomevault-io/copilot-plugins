## notch-rip

> Define interfaces for configuration and data structures:


# TypeScript Conventions

## Type Definitions

### Interface Definitions
Define interfaces for configuration and data structures:

```typescript
export interface Config {
  aspect: {
    height: number
    width: number
  }
  corners: {
    bl: boolean
    br: boolean
    tl: boolean
    tr: boolean
  }
  menubarHeightScale: number
  notch: boolean
}
```

### Const Assertions
Use `as const` for literal types and readonly arrays:

```typescript
export const PRESET_KEYS = ['MacBook', 'Desktop'] as const
export type PresetKey = typeof PRESET_KEYS[number]
```

### Generic Types
Use generic types for reusable patterns:

```typescript
import type { ClassValue } from 'clsx'
import type { VariantProps } from 'class-variance-authority'
```

## Vue Component Types

### Props and Emits
Always type props and emits explicitly:

```typescript
const props = defineProps<SliderRootProps & { class?: HTMLAttributes['class'] }>()
const emits = defineEmits<SliderRootEmits>()
```

### Template Refs
Use proper typing for template refs:

```typescript
const stageRef = useTemplateRef<Konva.Stage>('stageRef')
const imageRef = useTemplateRef<Konva.Image>('imageRef')
```

### Computed Properties
Type computed properties when needed:

```typescript
const stageNode = computed<Konva.Stage | undefined>(() => 
  stageRef.value?.getNode()
)
```

## External Library Types

### Konva Types
Import and use Konva types for canvas operations:

```typescript
import Konva from 'konva'

const notchRectConfig = computed<Konva.RectConfig>(() => ({
  cornerRadius: [0, 0, 9, 9],
  fill: 'black',
  height: getNotchHeight(canvasSize.value.height),
  // ...
}))
```

### Reka UI Types
Use Reka UI component types:

```typescript
import type { SliderRootEmits, SliderRootProps } from 'reka-ui'
```

## Error Handling

### Type Assertions
Use `@ts-expect-error` with comments for known type issues:

```typescript
// @ts-expect-error incorrect types
const stageNode = computed<Konva.Stage | undefined>(() => stageRef.value?.getNode())
```

### Optional Chaining
Use optional chaining for safe property access:

```typescript
const menubarHeight = computed(() => 
  (stageNode.value?.height() ?? 0) * config.menubarHeightScale.value
)
```

## Utility Types

### HTML Attributes
Use `HTMLAttributes` for DOM element props:

```typescript
const props = defineProps<ComponentProps & { class?: HTMLAttributes['class'] }>()
```

### Function Types
Define function types for callbacks:

```typescript
const { onCanvasDownloadTrigger } = useCanvas()
onCanvasDownloadTrigger(saveImage)
```

## Configuration Types

### Preset Types
Use mapped types for configuration presets:

```typescript
export const PRESETS: Record<PresetKey, Config> = {
  Desktop: { /* config */ },
  MacBook: { /* config */ },
} as const
```

### Cookie Types
Type cookie values appropriately:

```typescript
const corners = useCookie('config:corners', {
  default: () => ({ ...PRESETS.MacBook.corners }),
})
```

## ESLint Configuration

The project uses `@antfu/eslint-config` with:
- TypeScript-specific rules
- Vue-specific rules
- Import/export rules
- Formatting rules

Follow the established patterns for consistent code style.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jvxz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jvxz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
