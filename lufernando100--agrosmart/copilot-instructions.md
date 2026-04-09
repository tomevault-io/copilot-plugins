## agrosmart

> Antes de escribir cualquier línea de código, lee estos archivos en orden:

@AGENTS.md

---

## INICIO DE SESIÓN — LEER SIEMPRE PRIMERO

Antes de escribir cualquier línea de código, lee estos archivos en orden:

1. `PLAN.md` — estado actual de todas las tareas (qué está ✅ completo y qué está ⏳ pendiente)
2. `database/01_data_model.sql` — esquema completo de BD (tablas, columnas, relaciones, triggers)
3. `src/types/database.ts` — tipos TypeScript de todas las entidades
4. El archivo de wireframes o diseño relevante a la tarea (`docs/04_wireframes.md`, `docs/06_diseno_ui.md`)

Si la tarea toca lógica existente, lee también el archivo fuente antes de modificarlo.

---

## CODE LANGUAGE RULE

```
Everything a DEVELOPER sees  →  ENGLISH
Everything a USER sees       →  SPANISH (Colombian)
```

| Layer | Language | Examples |
|---|---|---|
| Variable and state names | ✅ English | `quantity`, `notes`, `isAvailable` |
| Function names | ✅ English | `createOrder()`, `sendWhatsAppMessage()` |
| TypeScript types and interfaces | ✅ English | `ProductSummary`, `OrderStatus`, `State` |
| Component props | ✅ English | `categories`, `unitPrice`, `activeCategoryId` |
| API JSON fields | ✅ English | `warehouse_id`, `action`, `confirmed_price` |
| File names | ✅ English | already correct |
| Code comments | ✅ English | `// Strip non-numeric chars from phone` |
| Button labels, placeholders, error messages | 🇨🇴 Spanish | `"Pedir aquí"`, `"Sin conexión…"` |
| AI system prompt and responses | 🇨🇴 Spanish | content for the farmer |

---

## COMANDOS

```bash
npm run dev      # Servidor de desarrollo en localhost:3000
npm run build    # Build de producción
npm run lint     # Linter ESLint
npm test         # Pruebas unitarias y de componentes (Vitest)
npm run test:watch  # Pruebas en modo watch
npm run ci       # Igual que GitHub Actions: lint + typecheck + test + build + E2E (env Supabase placeholder)
npm run test:e2e # Solo Playwright (levanta `next dev` solo)
supabase gen types typescript --local > src/types/supabase.ts  # Regenerar tipos tras cambios en BD
```

---

## ESTADO ACTUAL DEL PROYECTO
> Ver `PLAN.md` para el seguimiento detallado con estados por tarea

**Fase 1 — Marketplace: COMPLETADA ✅**
- Setup Next.js 16 con TypeScript, Tailwind, App Router
- Auth por OTP con teléfono + middleware de roles
- Catálogo de productos con comparador de precios (PostGIS) + QuickAdd
- Flujo de pedido completo (caficultor → almacén → confirmación con Realtime)
- Panel del almacén (dashboard, pedidos, productos/precios)
- Webhook de WhatsApp con Claude + tools (buscar_productos, crear_pedido, notificar_almacen)
- Notificaciones WhatsApp bidireccionales (almacén responde SI/NO)
- Diseño tierra/café, Plus Jakarta Sans, sidebar desktop + tab bar mobile
- 109 tests automatizados (Vitest + React Testing Library), 12 archivos

**Fase 2 — Inteligencia agronómica: EN CURSO**
- [ ] 2.1 Análisis de suelo con tablas Cenicafé + Claude Vision
- [ ] 2.2 Finca y lotes con mapa GPS (Google Maps)
- [ ] 2.3 Floraciones y alertas proactivas
- [ ] 2.4 Alertas climáticas con Open-Meteo

Ver el plan completo de 4 fases en `AGENTS.md` y `PLAN.md`.

---

## REGLAS RÁPIDAS

**Hacer:**
- **Idioma:** código, esquema de BD y nombres de API en **inglés**; textos de UI, WhatsApp de producto y errores al usuario en **español** (Colombia).
- TypeScript estricto siempre — cero `any`
- Usar `createClient()` de `lib/supabase/server.ts` en API routes y Server Components
- Usar `createAdminClient()` de `lib/supabase/admin.ts` solo para operaciones privilegiadas
- Commits pequeños en español: `feat: agrega comparador de precios`
- Responder 200 inmediato en el webhook de WhatsApp y procesar en background

**No hacer:**
- No modificar tablas de Supabase directamente en producción — usar migraciones
- No poner API keys en el código — todo en variables de entorno
- No construir Fase 2, 3 o 4 antes de tener un pedido real funcionando de punta a punta
- No usar `any` ni `as unknown` — definir tipos correctos

---

## VALIDACIONES Y MENSAJES DE ERROR — REGLAS OBLIGATORIAS

Estas reglas aplican a **todo código nuevo** que toque formularios, APIs o BD.

### Errores de base de datos
- **NUNCA** mostrar al usuario mensajes crudos de Postgres (códigos 23503, 23505, etc.)
- Usar siempre `friendlyDbError(err)` de `src/lib/utils/db-errors.ts` en el `catch` de operaciones de BD
- Antes de cualquier `INSERT` en `pedidos`: verificar que `caficultor_id` existe en `public.usuarios`
  ```ts
  const { data: cafRow } = await supabase.from('usuarios').select('id').eq('id', caficultorId).maybeSingle()
  if (!cafRow) throw new Error('Tu perfil no está listo. Cierra sesión, vuelve a entrar e intenta de nuevo.')
  ```

### Validaciones de campos (client + server)
| Campo | Regla |
|---|---|
| `notas` / textarea | `maxLength={500}`, siempre `.trim()` antes de guardar |
| `cantidad` | entero, `min=1`, `max=9999` |
| Cualquier ID | validar con `isUuid()` antes de usarlo |
| Texto libre | nunca mostrar en el UI sin sanitizar |

### Componentes de error
- Usar `<MensajeError message={...} />` de `src/components/ui/MensajeError.tsx`
- Siempre incluir `role="alert"` en mensajes de error (ya incluido en el componente)
- Para estados vacíos de listas, usar `<MensajeVacio />` de `src/components/ui/MensajeVacio.tsx`
- **Nunca** mostrar stack traces, nombres de tablas, ni códigos SQL al usuario

### Fotos de productos
- La foto se almacena en `metadata.foto_url` (JSONB) de la tabla `productos`
- En el UI, mostrar siempre placeholder si `foto_url` es null (inicial del nombre en bloque coloreado)
- Usar `<Image>` de next/image, nunca `<img>`, para optimización automática

### Tests negativos (obligatorios)
- Todo servicio que hace INSERT debe tener tests con FK violations
- Todo formulario debe tener tests con inputs inválidos (vacío, muy largo, negativo, tipo incorrecto)
- Los tests deben verificar que el **mensaje que ve el usuario** es amigable (en español, sin jerga técnica)

### Reglas operativas de Git (obligatorias)
- Ramas de feature cortas: abrir PR en 24-72h, no dejar ramas largas sin integrar
- Sincronizar rama activa con `main` al menos 1 vez al día mientras esté abierta
- Separar alcance por rama: cambios de diseño/UI no se mezclan con API/servicios/BD
- Antes de merge a `main`: ejecutar `npm test` y `npm run build`
- Si el merge presenta muchos conflictos o toca archivos críticos fuera del alcance: abortar y rehacer en PRs pequeños
- No desplegar nuevos cambios dejando merges conflictivos pendientes de ramas antiguas

### Depuración de auth (ambiente local)
Si ves `foreign key constraint "pedidos_caficultor_id_fkey"`:
1. Ve a Supabase → Authentication → Users y copia el UUID del usuario de prueba
2. Ejecuta: `SELECT id FROM public.usuarios WHERE id = '<UUID>'`
3. Si no hay fila: el OTP sync falló. Cierra sesión, borra cookies, y loguéate de nuevo
4. Si sí hay fila pero el error persiste: revisar RLS de la tabla `pedidos`

---

## ARCHIVOS CLAVE

| Archivo | Para qué sirve |
|---|---|
| `database/01_data_model.sql` | Esquema completo — leer antes de tocar la BD |
| `database/05_seed_data.sql` | Datos de ejemplo para desarrollo |
| `docs/02_system_prompt_tools.ts` | System prompt y 11 tools del asistente IA |
| `docs/03_flujos_whatsapp.md` | 8 flujos conversacionales completos |
| `docs/04_wireframes.md` | Todas las pantallas de la PWA |
| `docs/06_diseno_ui.md` | Especificación UI/UX: paleta, tipografía, componentes, responsive |
| `docs/PROMPT_COMPLETO.md` | Prompt de desarrollo completo con todo el contexto |
| `src/lib/cenicafe/tablas.ts` | Lógica de interpretación de análisis de suelo |
| `src/types/database.ts` | Tipos TypeScript de todas las entidades |
| `vitest.config.ts` | Configuración de pruebas (Vitest + jsdom) |
| `.env.local.example` | Variables de entorno requeridas |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lufernando100)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lufernando100)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
