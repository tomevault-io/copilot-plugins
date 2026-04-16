## ar

> UI component patterns and best practices for Alt Regioni

# Component Patterns

This rule outlines the component patterns and best practices followed in the Alt Regioni project.

## Component Structure

All Vue components follow the Vue 3 Composition API with `<script setup lang="ts">` syntax:

```vue
<script setup lang="ts">
// Imports
import { ref } from 'vue'
import { Button } from '@/components/ui/button'

// Props
interface Props {
  title: string
  description?: string
}

const props = defineProps<Props>()

// Emits
const emit = defineEmits<{
  (e: 'update', value: string): void
  (e: 'delete'): void
}>()

// State & computed
const isOpen = ref(false)

// Methods
function toggle() {
  isOpen.value = !isOpen.value
}
</script>

<template>
  <div>
    <!-- Component template -->
  </div>
</template>
```

## Component Organization

Components are organized into the following categories:

1. **UI Components** (`/src/client/components/ui/*`)
   - Base components following shadcn-vue patterns
   - Every component has its own directory with index.ts for exports

2. **Feature Components** (`/src/client/components/*`)
   - Specific domain components like:
     - `/organizations` - Organization-related components
     - `/users` - User-related components
     - `/forms` - Form-related components

3. **Layout Components** (`/src/client/layouts/*`)
   - Page layouts like `default.vue` and `admin.vue`

4. **Page Components** (`/src/client/pages/*`)
   - Full page components organized by routes

## shadcn-vue Usage

The project uses [shadcn-vue](https://shadcn-vue.com/) for UI components. Key patterns:

1. **Component Import**

```typescript
// Import from local ui component directory
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
```

2. **Composable Components**

Components are designed to be composed together:

```vue
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
    <CardDescription>Description</CardDescription>
  </CardHeader>
  <CardContent>
    <!-- Content here -->
  </CardContent>
  <CardFooter>
    <!-- Footer here -->
  </CardFooter>
</Card>
```

3. **Component Variants**

UI components support variants through class props:

```vue
<Button variant="outline" size="sm">Click me</Button>
```

## DataTable Pattern

DataTable components follow a specific pattern:

1. **Column Definition**

Files named `column.ts` define table columns:

```typescript
// Example from organizations/column.ts
export const columns = [
  {
    accessorKey: "name",
    header: ({ column }) => <ColumnHeader column={column} title="Name" />,
    cell: ({ row }) => {
      const name = row.getValue("name") as string
      return <div>{name}</div>
    },
  },
  // Other columns...
]
```

2. **DataTable Component**

```vue
<DataTable
  :columns="columns"
  :data="data"
  :filter-column="filterColumn"
  :filter-options="filterOptions"
/>
```

3. **Row Actions**

Row actions are defined in `RowActions.vue` components.

## Form Component Patterns

Forms use a combination of shadcn-vue components and Vue's reactivity:

```vue
<form @submit.prevent="onSubmit">
  <div class="space-y-4">
    <div class="space-y-2">
      <Label for="name">Name</Label>
      <Input id="name" v-model="form.name" />
      <p v-if="errors.name" class="text-sm text-red-500">
        {{ errors.name }}
      </p>
    </div>
    <!-- Other form fields -->
    <Button type="submit" :disabled="isSubmitting">Submit</Button>
  </div>
</form>
```

## Internationalization

Components support i18n through the `$t` function:

```vue
<h1>{{ $t('organizations.title') }}</h1>
```

## Best Practices

1. Use TypeScript interfaces for props and emits
2. Keep components focused on a single responsibility
3. Use composable components for complex UI
4. Extract reusable logic to composables
5. Leverage Tailwind CSS for styling
6. Follow PascalCase for component names
7. Use kebab-case for custom events
8. Organize imports with dependencies first, then local imports

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/semyenov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
