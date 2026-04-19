## inscripcions-nocturna

> Este proyecto es una aplicación para gestionar las **inscripciones de la Nocturna Fredes Paüls**, una marcha excursionista nocturna, construida con Laravel 12 e Inertia.js + Vue 3 + TypeScript.

# Contexto del Proyecto

Este proyecto es una aplicación para gestionar las **inscripciones de la Nocturna Fredes Paüls**, una marcha excursionista nocturna, construida con Laravel 12 e Inertia.js + Vue 3 + TypeScript.

## Objetivo

Sistema de gestión de inscripciones para la marcha excursionista Nocturna Fredes Paüls.

## Stack Principal

- Laravel 12 (backend)
- Vue 3 + TypeScript (frontend)
- Inertia.js (bridge Laravel-Vue)
- Tailwind CSS v4 + shadcn-vue (UI)
- MySQL 8.0 (base de datos)

## Patrones a Seguir

### Componentes Vue

```vue
<script setup lang="ts">
import { defineProps } from 'vue';

defineProps<{
  title: string;
  description?: string;
}>();
</script>

<template>
  <div>{{ title }}</div>
</template>
```

### Páginas Inertia

```vue
<script setup lang="ts">
import { Head } from '@inertiajs/vue3';

defineProps<{
  data: SomeType;
}>();
</script>

<template>
  <Head title="Página" />
  <!-- contenido -->
</template>
```

### Rutas Laravel

```php
Route::get('/', function () {
    return Inertia::render('Welcome', [
        'data' => $data
    ]);
});
```

## Configuración Importante

- TypeScript estricto habilitado
- ESLint + Prettier configurados
- Path alias `@/` apunta a `resources/js/`
- CSS usa variables CSS personalizadas para temas

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davidpru) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
