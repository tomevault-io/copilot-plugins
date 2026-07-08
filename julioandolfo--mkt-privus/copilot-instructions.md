## vue-inertia-conventions

> Convencoes Vue 3 + Inertia.js para o projeto MKT Privus


# Convencoes Vue 3 + Inertia.js

## Estrutura de Pastas (resources/js/)
- `Pages/` - Paginas Inertia (mapeiam para rotas Laravel)
  - `Dashboard/`, `Social/`, `Chat/`, `Blog/`, `Links/`, `Analytics/`, `Metrics/`, `Brands/`
- `Components/` - Componentes reutilizaveis
  - `UI/` - Componentes base (Shadcn-vue)
  - `Charts/` - Graficos (ECharts wrappers)
  - `Forms/` - Componentes de formulario
  - `Layout/` - Sidebar, Header, Navigation
- `Composables/` - Composables Vue (use*.ts)
- `Stores/` - Estado global se necessario (Pinia)
- `Types/` - TypeScript types/interfaces
- `Lib/` - Utilidades e helpers

## Padroes de Codigo
- Sempre usar `<script setup lang="ts">` em componentes Vue
- Usar Composition API (NUNCA Options API)
- Props devem ser tipadas com `defineProps<{...}>()`
- Emits devem ser tipados com `defineEmits<{...}>()`
- Usar Inertia `router` para navegacao (NUNCA vue-router)
- Usar Inertia `useForm()` para formularios com submissao
- Componentes de pagina recebem props do Controller Laravel

## Inertia.js Patterns
```vue
<script setup lang="ts">
import { Head, useForm, router } from '@inertiajs/vue3'

// Props vindas do Controller Laravel
const props = defineProps<{
  brands: App.Models.Brand[]
  currentBrand: App.Models.Brand
}>()

// Formulario Inertia
const form = useForm({
  name: '',
  description: '',
})

const submit = () => {
  form.post(route('brands.store'))
}
</script>
```

## UI e Estilo
- Tailwind CSS para estilizacao (NUNCA CSS puro ou SCSS inline)
- Shadcn-vue como base de componentes UI
- Tema escuro como padrao (dark mode)
- Interface em portugues Brasil
- Responsivo (mobile-first)
- Graficos com Apache ECharts via wrapper components

## Naming Conventions
- Paginas: PascalCase (`Social/Index.vue`, `Analytics/Dashboard.vue`)
- Componentes: PascalCase (`PostCard.vue`, `MetricChart.vue`)
- Composables: camelCase com prefixo use (`useBrand.ts`, `useChart.ts`)
- Types: PascalCase interfaces (`Post`, `Brand`, `MetricEntry`)

---
> Source: [julioandolfo/mkt-privus](https://github.com/julioandolfo/mkt-privus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
