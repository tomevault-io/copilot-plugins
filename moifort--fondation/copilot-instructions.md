## fondation

> [Front] Vue component best practice

# MDC Rules for storybook stories
These rules strictly apply to vue components

## General Rules
- A `my-componet.stories.ts` **MUST** be created for every component.
- If component it's a modal you **MUST** following template shown in Modal Implementation.

## Development Lifecycle Process
When developing validators, **YOU MUST FOLLOW** this step-by-step process:
1. **Analyze folder name and all files insides the other folder using the same name**  to improve you context. 
2. **Generate Code**: Implement my-component.vue following vue best practice and the patterns shown in Component Implementation
3. **Run Typecheck**: Run automatically run `bun typecheck` to validate yours types

## Modal Implementation
When implementing modal, follow this pattern:
- Implement `isOpen` as a `ref` and set it to `true` by default.
- Always use `close?: () => void` in Props
- When action closing the modal, set `isOpen` to `false` and call the `close` function.

```vue
<script setup lang="ts">
import { Button } from '~/components/atoms/button'
import { CardTitle } from '~/components/atoms/card'
import { AlertDialog, AlertDialogContent, AlertDialogHeader } from '~/components/atoms/alert-dialog'

defineProps<{ close?: () => void; price: number; }>()
const isOpen = ref(true)
</script>

<template>
  <AlertDialog :open="isOpen" >
    <AlertDialogContent class="rounded-xl">
        <AlertDialogHeader class="justify-center items-center mt-4">
          <slot name="icon" />

          <CardTitle class="text-4xl font-bold mb-4 mt-2">
            <slot name="title" />
          </CardTitle>
          <p class="text-2xl mb-2">The price was <span class="font-bold">${{ formatNumber(price) }}</span></p>
        </AlertDialogHeader>

        <AlertDialogFooter class="gap-3 flex-col">
          <Button
              variant="outline"
              class="w-full py-6 text-lg"
          >
            See Product
          </Button>

          <Button
              variant="default"
              class="w-full py-6 text-lg"
              @click="() => {
                isOpen = false
                close!()
              }"
          >
            Next Product
          </Button>
        </AlertDialogFooter>
    </AlertDialogContent>
  </AlertDialog>
</template>

```

Following these guidelines ensures validators remain clear, maintainable, and aligned with best DDD practices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/moifort)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/moifort)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
