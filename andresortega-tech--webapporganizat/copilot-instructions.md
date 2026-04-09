## webapporganizat

> cuando se modifique tarea, revisamos las reglas

# Reglas de Tasks

Este documento define el funcionamiento del módulo de tareas, incluyendo reglas de filtrado, ordenamiento y contratos de request/response. Mantenerlo actualizado con cualquier cambio en `tasks/api.py` y `tasks/schemas.py`.

## 1. Endpoints (Tasks)

### Crear tarea
**POST** `/api/tasks/`

**Request:**
```json
{
  "title": "Nueva tarea",
  "description": "Detalle opcional",
  "priority": "alta",
  "due_date": "2026-02-19T10:00:00Z",
  "reminders": [
    { "unit": "minutes", "value": 30 }
  ]
}
```

**Response (201 Created):**
```json
{
  "id": "uuid-task",
  "title": "Nueva tarea",
  "description": "Detalle opcional",
  "due_date": "2026-02-19T10:00:00Z",
  "is_completed": false,
  "priority": "alta",
  "user_id": "uuid-user",
  "calendar_event_id": "uuid-task",
  "media_url": null,
  "created_at": "2026-02-18T10:00:00Z",
  "updated_at": "2026-02-18T10:00:00Z",
  "reminders_data": [
    { "id": "uuid-reminder", "remind_at": "2026-02-19T09:30:00Z", "status": "pending" }
  ],
  "has_reminder": true
}
```

### Listar tareas (paginado)
**GET** `/api/tasks/`

**Query params:**
- `view`: `home` | `tasks`
- `tab`: `pending` | `completed` (solo en `view=tasks`, default `pending`)
- `tag_ids`: array de UUID (AND, solo en `view=tasks`)
- `priority`: `baja` | `media` | `alta` (solo en `view=tasks`)
- `end_date`: ISO 8601 (solo en `view=tasks`)
- `limit`: 1–50 (default 10)
- `cursor`: ISO 8601 (due_date del último item recibido)

**Response (200 OK):**
```json
{
  "data": [],
  "next_cursor": null,
  "has_more": false
}
```

### Reglas de `view=home`
- Ventana: `hoy` → `hoy + 7 días`.
- Se muestran:
  - Pendientes atrasadas.
  - Pendientes dentro de la ventana.
  - Completadas dentro de la ventana.
- No se muestran:
  - Completadas atrasadas.
  - Tareas sin `due_date`.
- Orden:
  - Pendientes atrasadas → `due_date` asc.
  - Pendientes futuras → `due_date` asc.
  - Completadas dentro de ventana → `due_date` asc, al final.

### Reglas de `view=tasks`
- Tab `pending`:
  - Solo tareas pendientes (`is_completed=false`).
  - Atrasadas primero (por `due_date` asc).
  - Luego futuras (`due_date` >= hoy, asc).
  - Tareas sin `due_date` al final.
- Tab `completed`:
  - Solo tareas completadas (`is_completed=true`).
  - Orden por `due_date` desc (más recientes primero).
  - Tareas sin `due_date` al final.

### Obtener tarea por ID
**GET** `/api/tasks/{id}`

**Response (200 OK):**
```json
{
  "title": "Integration Test Task",
  "description": "Testing full flow",
  "due_date": "2026-02-08T21:35:25.777257Z",
  "is_completed": false,
  "priority": "alta",
  "id": "uuid-task",
  "reminders_data": [],
  "has_reminder": false
}
```

### Obtener relaciones de una tarea
**GET** `/api/tasks/{id}/related`
 
**Response (200 OK):**
```json
{
  "tags": [
    { "id": "uuid-tag", "name": "Tag", "color": "#FF5733", "icon": "star" }
  ],
  "notes": [
    { "id": "uuid-note", "title": "Nota", "content": "Contenido completo" }
  ],
  "events": [
    { "id": "uuid-event", "title": "Evento", "start_time": "2026-02-20T10:00:00Z" }
  ]
}
```

> Las relaciones entre tareas, notas y eventos se crean y eliminan ahora desde el módulo `Relations`.

### Vincular etiqueta a tarea
**POST** `/api/tasks/{id}/tags`

**Request:**
```json
{ "tag_id": "uuid-tag" }
```

**Response (200 OK):**
```json
{ "message": "Etiqueta asignada correctamente", "assigned": 1 }
```

## 2. Notas de implementación
- `calendar_event_id` se iguala al `id` de la tarea al crearla.
- `reminders_data` se calcula desde `reminders` cuando hay `due_date`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AndresOrtega-tech)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AndresOrtega-tech)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
