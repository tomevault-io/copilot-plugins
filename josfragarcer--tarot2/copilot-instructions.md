## tarot2

> *(Versión completa — para uso en reglas globales de Windsurf)*


# 🧠 **TAROT2 — WINDSURF EDITING RULES**

### **Creative + Safe Mode**

*(Versión completa — para uso en reglas globales de Windsurf)*

---

# 0. **Modo de trabajo general**

**Tu rol:** actuar como un *ingeniero senior del proyecto Tarot2*, con libertad creativa para mejorar, refactorizar y extender el sistema, **pero siempre dentro de los límites definidos aquí**.

**NUNCA inventes APIs, componentes o rutas que no existan en:**

* el repo real
* la documentación oficial del MCP de **Nuxt**
* el MCP de **Nuxt UI**
* los **CodeMaps** del proyecto

**Siempre consulta antes de editar:**

1. **CodeMaps** (frontend architecture, backend API/dataflow, domain/entity model, UI patterns)
2. **MCP de Nuxt**
3. **MCP de Nuxt UI**
4. `SCHEMA POSTGRES..TXT` (fuente de verdad del modelo de datos)

---

# 1. **Reglas estructurales obligatorias**

## 1.1. No inventar

* No inventes nombres de props, slots, métodos o endpoints.
* Si necesitas un componente nuevo, debe seguir patrones existentes del repo o de MCP Nuxt UI (por ejemplo, duplicar patrones de `ManageTableBridge` o `FormModal`).

## 1.2. No romper invariantes centrales

Estas **jamás** deben romperse:

### A. Tabla unificada

Toda tabla debe usar:

* `CommonDataTable.vue`
* `ManageTableBridge.vue` (Manage)
* `AdminTableBridge.vue` (Admin)

**PROHIBIDO:** crear nuevas tablas con `<table>` o `UTable` directo (salvo casos documentados).

### B. Preview unificado

Toda previsualización de entidades debe usar:

* `EntityInspectorDrawer.vue`

**PROHIBIDO:** reintroducir `PreviewModal.vue` u otros modales custom.

### C. Formularios unificados

Toda creación/edición debe usar:

* `FormModal.vue`
* `entityFieldPresets` (o los nuevos presets declarativos)

**PROHIBIDO:** crear formularios ad-hoc en otros modales.

### D. Lógica CRUD unificada (backend)

Toda entidad Manage/Admin debe usar:

* `createCrudHandlers`

**PROHIBIDO:** crear endpoints CRUD manuales sin este patrón.

### E. Capabilities

Todo comportamiento condicional depende de:

* `useEntityCapabilities`

---

# 2. **Reglas de FRONTEND — Creative + Safe Mode**

## 2.1. Tablas y listados

### Debes usar:

* `CommonDataTable.vue`
* `ManageTableBridge.vue`
* `AdminTableBridge.vue`
* Selección con `useTableSelection`
* Bulk actions con `BulkActionsBar.vue`
* Columnas condicionales mediante capabilities

### Debes migrar (automáticamente si editas):

* `EntityTableWrapper.vue`
* `EntityTable.vue`
* Tablas de Admin como:

  * `VersionList.vue`
  * `RevisionHistory.vue`
  * `UserTable.vue` (legacy wrapper)

## 2.2. Previews

### Debes usar:

* `EntityInspectorDrawer.vue`
* preview perezoso vía `useEntityPreviewFetch`
* StatusBadge & tags dentro del drawer

## 2.3. Formularios

### Debes usar:

* `FormModal.vue`
* presets unificados (`useEntityFormPreset` si está implementado)

### Debes migrar:

* `RoleForm.vue`
* Modales de diff/snapshots en `RevisionHistory.vue`
* Cualquier modal que construya su propio formulario

## 2.4. Nuxt UI (reglas MCP)

### Debes:

* Validar cada uso de `U*` contra MCP de Nuxt UI
* Usar `v-model` / `v-model:prop`
* No usar APIs obsoletas como `:model-value`

No uses sintaxis vieja o props no documentadas.

## 2.5. Filtros y toolbars

### Debes usar:

* `ManageEntityFilters.vue`
* `AdvancedFiltersPanel.vue`

### Debes migrar:

* Filtros custom en Admin/Manage (ej: ManageUsers, VersionList)

---

# 3. **Reglas de BACKEND — Creative + Safe Mode**

## 3.1. Zonas críticas (NO tocar sin motivo)

* `createCrudHandlers.ts`
* `buildFilters.ts`
* `createPaginatedResponse.ts`
* `translatableUpsert.ts`
* `deleteLocalizedEntity.ts`
* Cualquier `_crud.ts` de:

  * arcana
  * world
  * facet
  * skill
  * base_card
  * world_card
* Editorial:

  * `/content_versions/*`
  * `/content_revisions/*`
  * `/content_feedback/*`
* Seguridad:

  * middleware `00.auth.hydrate`
  * `01.auth.guard`
  * `02.rate-limit`
  * `server/plugins/auth.ts`

## 3.2. Cambios permitidos con creatividad

Puedes:

* Extraer lógica duplicada (p.ej. filtros adicionales)
* Normalizar tag filtering
* Reforzar consistencia en fallback de traducción
* Añadir utilidades sin romper APIs
* Mejorar logs

## 3.3. Biblioteca de invariantes

Las **firmas de funciones** y **envolturas de respuesta** NO deben cambiar:

* `{ success, data, meta }`
* filtros deben seguir `buildFilters`
* respuesta con `createPaginatedResponse`
* SSR safe (`getUserFromEvent`, context.user)

---

# 4. **Reglas sobre BASE DE DATOS (SCHEMA POSTGRES)**

## 4.1. La DB real NO es accesible

Windsurf **no puede modificar el schema** real.
Debe basarse sólo en:

* `docs/SCHEMA POSTGRES..TXT`
* `server/database/types.ts`

## 4.2. Enums y dominios SON SAGRADOS

No inventar valores nuevos de:

* `card_status`
* `release_stage`
* `user_status`
* `feedback_status`
* dominios `entity_type`

## 4.3. Alta sensibilidad en:

* tablas `_translations`
* esquemas de efectos (`card_effects`)
* `tag_links`
* world_card overrides

---

# 5. **Reglas de DISEÑO CREATIVE + SAFE MODE**

## 5.1. Libertad creativa permitida cuando:

* mejoras UI
* unificas flujos
* migras legacy a patrones modernos
* creas utilidades que reducen duplicación
* refactorizas para claridad sin romper contratos
* mejoras accesibilidad

## 5.2. Libertad limitada cuando:

* tocas pipelines Manage/Admin
* tocas EntityBase.vue
* tocas bridges/table core
* tocas composición del formulario
* tocas lógicas editoriales o auth

---

# 6. **Migraciones automáticas permitidas**

**Si estás editando** un archivo que todavía usa un patrón viejo, Windsurf **DEBE** sugerir o aplicar migración a:

* CommonDataTable
* ManageTableBridge / AdminTableBridge
* EntityInspectorDrawer
* FormModal + presets
* StatusBadge unificado
* Filtros estándar

Ejemplos:

* Abrir `UserTable.vue` → sugerir migración a AdminTableBridge
* Abrir `VersionList.vue` → sugerir migración a AdminTableBridge
* Abrir `PreviewModal.vue` → sugerir reemplazar por drawer
* Abrir una tabla con HTML → sugerir CommonDataTable

---

# 7. **Checklist previo a confirmar cambios**

Antes de completar un PR, revisa:

### ✔ ¿Respeta invariantes centrales?

### ✔ ¿Usa patrones modernos del proyecto?

### ✔ ¿Consulta CodeMaps + MCP?

### ✔ ¿Mantiene SSR / i18n safe?

### ✔ ¿Respeta SCHEMA POSTGRES?

### ✔ ¿No inventa APIs?

### ✔ ¿Mantiene paridad funcional?

### ✔ ¿Ha migrado lo legacy dentro del archivo?

### ✔ ¿Ha evitado tocar zonas críticas sin motivo?

---

# 8. **Áreas explícitamente autorizadas para “mejora creativa guiada”**

* Migrar tablas legacy → bridges
* Migrar modales → FormModal / Drawer
* Crear toolbar unificada
* Crear diff modal unificado basado en JsonModal
* Añadir presets de formulario por entidad
* Estandarizar filtros
* Extraer lógica repetida del backend que no cambie contratos
* Limpiar imports, dead code, duplicaciones

---

# 9. **Áreas donde NO debe actuar salvo orden explícita**

* Modificar CRUD `_crud.ts` de entidades ya estables
* Tocar lógica editorial de `publish`, `revert`, `approve`
* Cambiar middleware de seguridad
* Cambiar shape de efectos JSON
* Cambiar enums de dominio
* Cambiar shape de respuesta API
* Cambiar pagination/filter contract

---

# 10. **Instrucción final para Windsurf**

> **Siempre que realices cualquier cambio, consúltalo primero con:
> CodeMaps + MCP Nuxt + MCP Nuxt UI + SCHEMA POSTGRES.**
>
> **Si un componente o endpoint contradice estos patrones, migra hacia la arquitectura moderna de Tarot2 de forma incremental, segura y con paridad funcional.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JosFraGarCer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
