## sistema-escolar-sql

> Sistema relacional completo de gestión académica escolar y universitaria en **SQLite**. Modela estudiantes, profesores, cursos, secciones, matrículas, evaluaciones, calificaciones y asistencias con integridad referencial estricta.

# Sistema de Gestión Escolar - Guía para Agentes de IA

## 🎯 Resumen del Proyecto

Sistema relacional completo de gestión académica escolar y universitaria en **SQLite**. Modela estudiantes, profesores, cursos, secciones, matrículas, evaluaciones, calificaciones y asistencias con integridad referencial estricta.

**Stack**: Python 3 + SQLite 3 + SQL (DDL/DML)

---

## 📁 Estructura de Archivos

| Archivo | Propósito | Mantenido por |
|---------|-----------|--------------|
| [`schema.sql`](schema.sql) | Definición de 11 tablas, índices, vistas y triggers | Equipo de base de datos |
| [`seed.sql`](seed.sql) | Datos de prueba iniciales (estudiantes, profesores, cursos, etc.) | Equipo de QA |
| [`queries.sql`](queries.sql) | Consultas de ejemplo y reportes frecuentes | Documentación |
| [`init_db.py`](init_db.py) | Script Python para inicializar `escuela.db` | Automatización |
| [`README.md`](README.md) | Documentación general y modelo E-R | Documentación |

---

## 🗂️ Modelo de Datos

### Tablas Principales (Orden de Dependencia)

1. **`departamentos`** — Facultades/áreas académicas
   - Referenciado por: `profesores`, `cursos`

2. **`profesores`** — Docentes con especialidad y departamento
   - FK: `departamento_id → departamentos`

3. **`estudiantes`** — Alumnos con matrícula única
   - Estados: `ACTIVO`, `INACTIVO`, `GRADUADO`, `SUSPENDIDO`

4. **`periodos_academicos`** — Semestres/trimestres (ej: "2026-1")
   - Estados: `PLANIFICACION`, `ACTIVO`, `FINALIZADO`

5. **`aulas`** — Salones con capacidad y tipo
   - Tipos: `TEORICA`, `LABORATORIO`, `AUDITORIO`

6. **`cursos`** — Asignaturas con código, créditos y descripción
   - FK: `departamento_id → departamentos`

7. **`secciones`** — Instancia de un curso en un período específico
   - FK: `curso_id`, `profesor_id`, `periodo_id`, `aula_id`
   - Único: `(curso_id, periodo_id, codigo_seccion)`

8. **`matriculas`** — Inscripción de estudiante en una sección
   - Estados: `INSCRITO`, `RETIRADO`, `APROBADO`, `REPROBADO`
   - Único: `(estudiante_id, seccion_id)`

9. **`evaluaciones`** — Parciales, tareas, proyectos por sección
   - FK: `seccion_id → secciones`
   - Rango de peso: 0-100%

10. **`calificaciones`** — Notas (0-100) por estudiante y evaluación
    - FK: `matricula_id`, `evaluacion_id`
    - Único: `(matricula_id, evaluacion_id)`

11. **`asistencias`** — Control diario de asistencia
    - Estados: `PRESENTE`, `AUSENTE`, `JUSTIFICADO`, `TARDANZA`
    - Único: `(matricula_id, fecha)`

---

## 🚀 Inicialización y Ejecución

### Método 1: Python (Recomendado)
```bash
cd sistema_escolar_sql
python init_db.py
```
Crea `escuela.db` limpia y ejecuta schema + seed automáticamente.

### Método 2: SQLite CLI (Manual)
```bash
sqlite3 escuela.db < schema.sql
sqlite3 escuela.db < seed.sql
```

### Método 3: Cliente Gráfico
Abre `escuela.db` directamente con DBeaver, DB Browser for SQLite, o VSCode con extensión SQLite.

---

## 🔑 Convenciones y Prácticas

### Nombres de Columnas y Tablas
- **Tablas**: plural en minúsculas (`estudiantes`, `calificaciones`)
- **Columnas**: snake_case (`documento_identidad`, `peso_porcentaje`)
- **IDs primarios**: `id` (INTEGER PRIMARY KEY AUTOINCREMENT)
- **FKs**: `{tabla_singular}_id` (ej: `estudiante_id`, `seccion_id`)
- **Timestamps**: `creado_en`, `fecha_registro` (DATETIME DEFAULT CURRENT_TIMESTAMP)

### Integridad Referencial
- **PRAGMA foreign_keys = ON;** — Habilitada en `schema.sql`
- **ON DELETE CASCADE** — Para matrículas, calificaciones y asistencias
- **ON DELETE RESTRICT** — Para profesores (no se pueden eliminar si tienen secciones)
- **ON DELETE SET NULL** — Para aulas y departamentos (opcional)

### CHECK Constraints
- Estados: restricción a valores específicos (`ACTIVO`, `INACTIVO`, etc.)
- Capacidades y cupos: > 0
- Notas: 0.0 a 100.0
- Pesos de evaluaciones: 0 < peso <= 100
- Fechas: `fecha_fin > fecha_inicio`

### UNIQUE Constraints
- Matrículas de estudiantes
- Documentos de profesores
- Emails (estudiantes y profesores)
- Códigos de departamentos, aulas, cursos
- `(estudiante_id, seccion_id)` para matrículas
- `(matricula_id, evaluacion_id)` para calificaciones
- `(matricula_id, fecha)` para asistencias
- `(curso_id, periodo_id, codigo_seccion)` para secciones

---

## 📊 Vistas Útiles

### `v_detalle_secciones`
Detalle completo de secciones con curso, profesor, aula, periodo, inscritos y cupos disponibles.

### `v_boleta_calificaciones`
Boleta de notas por estudiante con nota final ponderada (cálculo automático basado en pesos de evaluaciones).

**Nota**: Consultar estas vistas para reportes es más eficiente que joins manuales.

---

## ⚡ Patrones Comunes

### Crear una Nueva Sección
1. Verificar que existe `periodo_id` con estado `ACTIVO` o `PLANIFICACION`
2. Verificar que `curso_id` y `profesor_id` son válidos
3. Generar `codigo_seccion` único para ese período
4. Insertar en `secciones` con `cupo_maximo` y `horario`
5. Usuarios pueden inscribirse a través de `matriculas`

### Registrar Evaluaciones y Calificaciones
1. Crear evaluación en `evaluaciones` (asignar peso % que sume 100 por sección)
2. Por cada estudiante inscrito (`matriculas`), insertar fila en `calificaciones`
3. La nota final se calcula automáticamente en `v_boleta_calificaciones`

### Consultar Promedio de un Estudiante
```sql
SELECT nota_final_calculada 
FROM v_boleta_calificaciones 
WHERE matricula_estudiante = 'E12345';
```

### Verificar Disponibilidad de una Sección
```sql
SELECT cupos_disponibles 
FROM v_detalle_secciones 
WHERE seccion_id = ?;
```

---

## ⚠️ Consideraciones Importantes

1. **Cascada de eliminaciones**: Borrar una sección elimina todas sus matrículas, calificaciones y asistencias.
2. **Profesores no son eliminables**: Si intenta eliminar un profesor con secciones activas, fallará.
3. **Aulas opcionales**: Una sección puede tener `aula_id = NULL` (clase virtual).
4. **Período académico es crítico**: Todas las secciones dependen de un período. Planifique períodos al inicio.
5. **Estado de matrículas**: Es importante diferenciar entre `INSCRITO`, `RETIRADO`, `APROBADO`, `REPROBADO` para reportes.

---

## 📝 Tareas Comunes para Agentes

### Cuando alguien pide "agregar X funcionalidad":
- Identifique qué tabla(s) requiere modificar o crear
- Verifique la integridad referencial: ¿qué tablas la referencian?
- Actualice `schema.sql`, luego `seed.sql` con datos de prueba
- Documente cambios en este archivo
- Ejecute `python init_db.py` para validar

### Cuando alguien pide "reportar Y información":
- Primero intente usar las vistas existentes en `schema.sql`
- Si no existen, considere crear una nueva vista
- Consulte `queries.sql` para ejemplos similares
- Valide con datos del `seed.sql`

### Cuando el esquema cambiar:
- Edite `schema.sql` **antes** que `seed.sql`
- El script `init_db.py` elimina y recrea `escuela.db` cada ejecución
- No hay necesidad de migraciones; siempre es inicialización limpia

---

## 🔗 Referencias Rápidas

- [README del Proyecto](README.md) — Descripción general y modelo E-R
- [schema.sql](schema.sql) — Definición de tablas, índices y vistas
- [seed.sql](seed.sql) — Datos de prueba
- [queries.sql](queries.sql) — Ejemplos de consultas comunes
- [init_db.py](init_db.py) — Automatización en Python

---

## 💡 Consejos para Agentes de IA

1. **Siempre verificar PRAGMA foreign_keys antes de importar datos**
2. **Los pesos de evaluaciones en una sección deben sumar 100** (agregar validación si es necesario)
3. **Usar transacciones para operaciones complejas** (`BEGIN TRANSACTION; ... COMMIT;`)
4. **Los códigos (curso, seccion, aula, etc.) deben ser mnemotécnicos** (ej: `PROG201`, `LAB-SIS-1`)
5. **Consultar siempre las vistas para reportes complejos** antes de escribir queries manuales

---
> Source: [Adonis1510/sistema-escolar-sql](https://github.com/Adonis1510/sistema-escolar-sql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
