## 03-database

> Reglas para cambios en la base de datos MySQL — migraciones, nomenclatura y relaciones.


# Base de Datos — Reglas MySQL

## Migraciones
- **Obligatorio:** Cualquier cambio estructural (nueva tabla, columna, índice, tipo de dato) debe crearse como un archivo SQL numerado y secuencial en `backend/database/migrations/`.
  - Formato del nombre: `NNN_descripcion_corta.sql` (ej. `005_add_orden_to_temarios.sql`)
  - El número debe ser secuencial respecto al último archivo existente.
- Nunca modificar archivos de migración existentes — solo agregar nuevos.
- Nunca alterar la base de datos productiva directamente fuera de una migración.

## Nomenclatura
- Tablas: `snake_case`, plural (ej. `ciclos_escolares`, `dias_inhabiles`).
- Columnas: `snake_case` (ej. `fecha_inicio`, `token_balance`).
- FK: `{tabla_referenciada_singular}_id` (ej. `user_id`, `ciclo_id`).
- Índices: `idx_{columnas}` (ej. `idx_user_course`).

## Tablas principales y sus relaciones
```
perfiles (docente)
└── ciclos_escolares (periodo semestral/anual)
    └── ciclos (unidades temáticas → topics en Classroom)
        └── temarios (temas individuales)
    └── planeacion_detallada (clase por clase, temporal)
    └── dias_inhabiles

perfiles
└── materiales_generados (docs/slides en Drive)
└── actividades_evaluables (assignments con rubrica JSON)
└── horarios
└── unidades_fechas
```

## Reglas de borrado
- Al eliminar un `ciclo_escolar` → borrar en cascada sus `ciclos`, `planeacion_detallada`, y `dias_inhabiles`.
- Al eliminar un `ciclo` (unidad) → borrar su `classroom_topic_id` de Google Classroom y sus `temarios`.
- Al eliminar una `actividad_evaluable` → eliminar el `courseWork` en Google Classroom.
- Usar `ON DELETE CASCADE` en FK donde aplique, y lógica explícita en el servicio para operaciones en Google APIs.

## SQL en servicios
- Usar **siempre** parámetros preparados de `mysql2`:
```js
// ✅ Correcto
await db.query('SELECT * FROM perfiles WHERE google_id = ?', [googleId]);

// ❌ Incorrecto — inyección SQL
await db.query(`SELECT * FROM perfiles WHERE google_id = '${googleId}'`);
```
- Usar transacciones (`BEGIN` / `COMMIT` / `ROLLBACK`) para operaciones que involucren múltiples escrituras.

---
> Source: [MiguelAngelMondragonFernandez/Classroom-copilot](https://github.com/MiguelAngelMondragonFernandez/Classroom-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
