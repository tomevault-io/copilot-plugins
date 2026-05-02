## kanban-pro

> Cuando la petición del usuario empiece por **"crea una nueva funcionalidad"**, además de implementar el cambio:

# Codex â€” Kanban Pro

## Registro de funcionalidades

Cuando la petición del usuario empiece por **"crea una nueva funcionalidad"**, además de implementar el cambio:

1. Registra la nueva funcionalidad en `docs/FUNCIONALIDADES_Y_REGISTRO.md`
2. Añade una nueva entrada usando la plantilla de "Registro de nuevas funcionalidades"
3. Describe la funcionalidad y su funcionamiento real, no una intención genérica
4. Incluye al menos objetivo, funcionamiento, reglas de negocio, impacto en usuario y archivos principales

Si la petición del usuario **modifica un comportamiento existente**:

1. Revisa `docs/FUNCIONALIDADES_Y_REGISTRO.md`
2. Verifica si la funcionalidad afectada ya está documentada
3. Actualiza la descripción funcional o el funcionamiento si el comportamiento ha cambiado
4. Antes de cerrar la tarea, comprueba que `docs/FUNCIONALIDADES_Y_REGISTRO.md` queda alineado con el comportamiento final implementado

No des por terminada una tarea que cree o altere funcionalidades sin revisar ese documento.

## Aplicar mejoras pendientes

Cuando el usuario escriba **"aplica las mejoras pendientes"**, sigue estos pasos:

### 1. Verificar que existe el archivo de mejoras

Comprueba si existe `MEJORAS_PENDIENTES.md` en la raíz del proyecto.

**Si NO existe**, detente y muestra este mensaje de error:

> ❌ No se encontró el archivo `MEJORAS_PENDIENTES.md`.
> Para generarlo, ve a la pantalla **MEJORAS** de la aplicación, selecciona las mejoras pendientes y pulsa el botón **APLICAR**. El archivo se creará automáticamente en la raíz del proyecto.

**Si existe**, lee su contenido y continúa con los siguientes pasos.

### 2. Analizar y aplicar cambios

Por cada mejora:
- Entiende qué cambio de UI, lógica o datos es necesario
- Localiza los archivos afectados en `src/`
- Aplica el cambio mínimo necesario
- No refactorices código no relacionado

### 3. Marcar como aplicadas en Supabase

Construye un resumen de los cambios y ejecuta:

```bash
export $(grep -v '^#' .env.local 2>/dev/null || grep -v '^#' .env 2>/dev/null | xargs)
curl -s -X PATCH "${VITE_SUPABASE_URL}/rest/v1/improvements?status=eq.ai_pending" \
  -H "apikey: ${VITE_SUPABASE_ANON_KEY}" \
  -H "Authorization: Bearer ${VITE_SUPABASE_ANON_KEY}" \
  -H "Content-Type: application/json" \
  -d "{\"status\":\"applied\",\"applied_at\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",\"ai_result\":\"[Resumen de los cambios aplicados]\"}"
```

Sustituye `[Resumen de los cambios aplicados]` por una descripción real de los cambios.

### 4. Limpiar (opcional)

Si existe `MEJORAS_PENDIENTES.md` en la raíz, elimínalo.

---

## Estructura del proyecto

```
src/
  App.tsx                        â€” Orquestador principal
  types.ts                       â€” Interfaces TypeScript
  constants.ts                   â€” Constantes globales
  hooks/useTheme.ts              â€” Tema reactivo (dark/light)
  lib/supabase.ts                â€” Cliente Supabase
  lib/db.ts                      â€” CRUD Supabase
  lib/utils.ts                   â€” Utilidades puras
  components/
    KCard.tsx                    â€” Tarjeta Kanban
    CardModal.tsx                â€” Modal de edición de tarjeta
    JustifyModal.tsx             â€” Modal de justificación
    UserModal.tsx                â€” Modal de nuevo usuario
    NewKanbanModal.tsx           â€” Modal de nuevo tablero
    ImprovementBtn.tsx           â€” Botón bombilla 💡
    ImprovementModal.tsx         â€” Modal para proponer mejora
    ImprovementsPage.tsx         â€” Página de gestión de mejoras
    settings/SettingsPage.tsx    â€” Página de configuración
    ui/                          â€” Componentes primitivos
```

## Tabla improvements en Supabase

Si la tabla `improvements` no existe aún, créala en el dashboard de Supabase:

```sql
CREATE TABLE improvements (
  id TEXT PRIMARY KEY,
  board_id TEXT NOT NULL,
  user_id TEXT NOT NULL,
  user_name TEXT NOT NULL,
  description TEXT NOT NULL,
  context TEXT NOT NULL DEFAULT 'board',
  status TEXT NOT NULL DEFAULT 'pending',
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  applied_at TIMESTAMPTZ,
  ai_result TEXT
);
```

---
> Source: [bakero/kanban-pro](https://github.com/bakero/kanban-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
