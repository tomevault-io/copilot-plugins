## fsi-monitor-yottagraph

> Apply when creating or editing feature page templates, layouts, scrollable content, data tables, or loading states in Vue/Vuetify components.


# UI Patterns

## Vuetify Layout System

Do not modify the core layout in `app.vue`.

- Never remove or override `v-main` padding
- Always use `app` prop on `AppHeader` and `ModularSideNavPanel`
- Use `fill-height` class on containers that need full height
- Use Vuetify components (`v-card`, `v-btn`, `v-data-table`) over custom implementations
- Use Vuetify spacing utilities (`pa-4`, `ma-2`) and grid system (`v-row`, `v-col`)

## Feature Page Template

Every feature page must use `FeatureHeader` with a flexbox scroll layout:
- `d-flex flex-column` on the column container
- `flex-shrink-0` on fixed elements (header, toolbar)
- `flex-grow-1 overflow-y-auto` on scrollable content
- Never use `calc(100vh - Xpx)` -- let flexbox handle sizing
- Never nest multiple scroll containers

See `patterns/feature-page.vue` for the full template.

## Loading States

```vue
<v-progress-circular v-if="loading" indeterminate />
<div v-else>
    <!-- Content -->
</div>
```

## Data Tables

```vue
<v-data-table :headers="headers" :items="items" :loading="loading" density="comfortable" hover>
    <template v-slot:item.actions="{ item }">
        <v-btn icon size="small" @click="selectItem(item)">
            <v-icon>mdi-eye</v-icon>
        </v-btn>
    </template>
</v-data-table>
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Lovelace-AI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
