## github-copilot-skill-prddiario

> Gestiona tareas diarias con formato jerárquico legible, documentación completa (descripción + solución) con timestamps, y genera reportes automáticos de horas trabajadas. Usa cuando necesites crear PRD diario, registrar tareas completadas, gestionar pendientes, o generar reportes de horas. Incluye scripts Python/PowerShell para automatizar todo.


# PRD Diario - Gestor de Tareas Diarias

## Descripción General

Automatiza la creación y organización de Documentos de Requisitos de Productos (PRD) diarios para un seguimiento estructurado de tareas. **Mantiene separadas las carpetas de PRD, trabajo diario, reportes y archivos**. Ideal para:

- **Organización flexible** - Configura dónde guardar cada tipo de documento
- **Carpetas PRD centralizadas** - Todos los `PRD_YYYYMMDD.md` en un solo lugar
- **Trabajo diario organizado** - Carpetas YYMMDD separadas para documentos del día
- **Reportes consolidados** - Resúmenes y análisis en una carpeta dedicada
- **Archivo histórico** - Días completados archivados para referencia
- **Registro de tareas** con timestamps exactos y documentación completa
- **Resúmenes automáticos** que analizan metadatos de archivos
- **Rastreo de inicio de jornada** usando fecha de creación de documentos
- **Reportes de horas trabajadas** y auditoría completa

## Cuándo Usar Este Skill

Use este skill cuando:

- Necesite **iniciar el día** y crear la carpeta + PRD diario
- Quiera **registrar tareas** completadas con hora exacta  
- Deba **documentar soluciones** de manera estructurada
- Tenga **tareas pendientes** que requieran seguimiento
- Necesite **guardar conversaciones** o documentos del día en un solo lugar
- Quiera **generar resumen del día** automático con metadatos
- Necesite **reportes de horas** trabajadas al final del día
- Desee saber **cuándo empezó el día** (primer documento creado)
- El usuario pida "crear PRD diario", "iniciar día", "registrar tarea", "resumen del día" o "generar reporte de horas"

## Flujo de Trabajo

### Fase 0: Configuración Inicial (IMPORTANTE!)

**Primera ejecución - Configura tus carpetas:**

```bash
python scripts/setup_config.py
```

Este wizard interactivo te permite elegir dónde guardar:
- 📄 **PRD_DOCUMENTS**: Donde van todos los `PRD_YYYYMMDD.md`
- 📁 **DAILY_WORK**: Donde se crean carpetas YYMMDD por día
- 📊 **REPORTS**: Donde se generan resúmenes y reportes
- 📦 **ARCHIVES**: Donde archivar días completados (opcional)

**Estructura recomendada:**
```
~/Documents/prd_diarios/
  ├── PRD_DOCUMENTS/              ← Todos los PRD centralizados
  │   ├── PRD_20260225.md
  │   ├── PRD_20260226.md
  │   └── PRD_20260227.md
  ├── DAILY_WORK/                 ← Trabajo diario por carpetas
  │   ├── 260225/
  │   │   ├── README.md
  │   │   ├── conversacion_cliente.md
  │   │   └── notas_meeting.md
  │   ├── 260226/
  │   └── 260227/
  ├── REPORTS/                    ← Resúmenes y reportes
  │   ├── RESUMEN_260225.md
  │   ├── RESUMEN_260226.md
  │   ├── HORAS_PRD_20260225.md
  │   └── HORAS_PRD_20260226.md
  └── ARCHIVES/                   ← Días completados
      └── (semanas antiguas)
```

**Ver configuración actual:**
```bash
python scripts/setup_config.py --show
```

**Resetear configuración:**
```bash
python scripts/setup_config.py --reset
```

### Fase 1: Inicio del Día (NUEVO)

Mientras trabajas:

1. **Registrar tareas**: Añade tareas completadas con timestamps
2. **Guardar documentos**: Guarda conversaciones, notas, archivos en la carpeta del día
3. **Actualizar PRD**: Documenta descripción + solución de cada tarea

**Ejemplo de estructura durante el día:**
```
260225/
  ├── README.md
  ├── PRD_20260225.md
  ├── conversacion_cliente_proyecto_X.md
  ├── notas_meeting_equipo.md
  └── diagrama_arquitectura.png
```

Mientras trabajas:

1. **Registrar tareas**: Añade tareas completadas con timestamps en el PRD
2. **Guardar documentos**: Guarda conversaciones, notas, archivos en DAILY_WORK/260225/
3. **Actualizar PRD**: Documenta descripción + solución de cada tarea en PRD_DOCUMENTS/

**Estructura típica durante el día:**
```
DAILY_WORK/260225/
  ├── README.md
  ├── conversacion_cliente_proyecto_X.md
  ├── notas_meeting_equipo.md
  └── diagrama_arquitectura.png

PRD_DOCUMENTS/
  └── PRD_20260225.md          ← Actualizado con tareas
```

### Fase 3: Fin del Día

Al terminar la jornada:

1. **Generar resumen del día**: Ejecuta `generate_day_summary.py`
2. **Analiza metadatos**: Lee fechas de creación de todos los archivos
3. **Calcula horas**: Determina inicio (primer archivo) y tareas realizadas
4. **Crea reporte**: Genera RESUMEN_YYMMDD.md con toda la información

**Estructura final:**
```
260225/
  ├── README.md
  ├── PRD_20260225.md
  ├── conversacion_cliente_proyecto_X.md
  ├── notas_meeting_equipo.md
  ├── diagrama_arquitectura.png
  ├── RESUMEN_260225.md          # ← Generado automáticamente
  └── HORAS_PRD_20260225.md      # ← Opcional: reporte de horas detallado
```

### Fase 3 (Antigua): Registrar Tareas Realizadas

Para cada tarea completada:

```markdown
### ✅ N. Nombre de la Tarea — **HH:MM**

**Descripción**  
Contexto y motivo de la tarea. Qué problema se resolvía, de dónde venía la solicitud.

**Solución**  
Qué se hizo y cómo se resolvió. Pasos tomados, tecnologías usadas, resultado final.
```

**Ejemplo:**

```markdown
### ✅ 1. Revisar tareas asignadas en Trello — **09:00**

**Descripción**  
Morning standup: Revisión de tareas pendientes del sprint. Se identificaron 12 tareas en el backlog y 3 en progreso.

**Solución**  
Se revisaron prioridades con el equipo. Se replanificó una tarea de baja prioridad. Se inició trabajo en tarea crítica de cliente.
```

### Fase 3: Gestionar Tareas Pendientes

Para tareas incompletas:

```markdown
### ⏳ X. Nombre de Tarea Pendiente — **HH:MM**

**Descripción**  
Contexto de la tarea pendiente...

**Estado**  
En curso / Bloqueado / En espera
```

### Fase 4: Generar Reportes de Horas

Al final del día, para generar un reporte automático:

**Python:**
```bash
python scripts/generate_hours_report.py PRD_260216.md
```

**PowerShell:**
```powershell
.\scripts\generate_hours_report.ps1 -PRDFile "PRD_260216.md"
```

Genera automáticamente `HORAS_PRD_260216.md` con:
- Desglose por tarea con duración
- Horas totales trabajadas
- Promedio por tarea
- Timestamp de generación

## Resumen Ejecutivo

- **Fecha**: DD de MMMM de YYYY
- **Tareas completadas**: N
- **Tareas pendientes**: M
- **Total de horas**: Xh YYm

## Tareas Realizadas

### ✅ 1. Primera Tarea — **09:00**

**Descripción**  
...

**Solución**  
...

### ✅ 2. Segunda Tarea — **10:30**

**Descripción**  
...

**Solución**  
...

## Tareas Pendientes

### ⏳ 1. Tarea Pendiente — **16:00**

**Descripción**  
...

**Estado**  
En curso

## Notas Adicionales

- Observaciones importantes
```

## Patrón de Uso - Paso a Paso

### ⚙️ Primer Día - Configuración Inicial

```
Usuario: "Configura el skill prddiario"
Claude:
1. Ejecuta: python scripts/setup_config.py
2. Pregunta interactiva por rutas:
   - ¿Dónde guardar los PRD?
   - ¿Dónde guardar el trabajo diario?
   - ¿Dónde guardar los reportes?
   - ¿Dónde archivar días completados?
3. Pregunta sobre features (resúmenes automáticos, metadata, etc.)
4. Guarda configuración en config.json
5. Confirma:
   ✅ PRD_DOCUMENTS configurado
   ✅ DAILY_WORK configurado
   ✅ REPORTS configurado
   ✅ ARCHIVES configurado
```

### Iniciar el Día

```
Usuario: "Vamos a iniciar el día" o "Crear PRD de hoy"
Claude:
1. Ejecuta: python scripts/create_daily_folder.py
   → Crea 260225 en DAILY_WORK/
2. Ejecuta: python scripts/create_daily_prd.py
   → Crea PRD_20260225.md en PRD_DOCUMENTS/
3. Confirma:
   ✅ Carpeta creada: ~/Documents/prd_diarios/DAILY_WORK/260225/
   ✅ PRD creado: ~/Documents/prd_diarios/PRD_DOCUMENTS/PRD_20260225.md
```

**Estructura después:**
```
DAILY_WORK/260225/          ← Nuevas conversaciones aquí
  └── README.md

PRD_DOCUMENTS/
  └── PRD_20260225.md       ← Actualizar con tareas del día
```

### Guardar Documentos Durante el Día

```
Usuario: "Guarda esta conversación en el día de hoy"
Claude:
1. Identifica fecha actual (ej: 260225)
2. Guarda archivo en: DAILY_WORK/260225/
3. El archivo queda organizado junto a otros documentos del día
```

**Ejemplos de archivos guardados:**
```
DAILY_WORK/260225/
  ├── README.md
  ├── conversacion_proyecto_X_cliente.md    ← Nuevo
  ├── notas_meeting_equipo.md              ← Nuevo
  └── diagrama_arquitectura.png             ← Nuevo
```

### Crear PRD Nuevo (Método Original)

```
Usuario: "Vamos a crear el PRD de hoy"
Claude:
1. Obtiene fecha actual (ej: 16 de febrero de 2026)
2. Si use_daily_folders=true, crea carpeta 260216
3. Crea PRD_20260216.md dentro de la carpeta (o en ruta especificada)
4. Abre el archivo para edición
```

### Registrar Tarea Completada

```
Usuario: "Completé: Revisar correos. Tomó 45 minutos. Fueron 23 correos, respondí prioritarios."
Claude:
1. Obtiene hora actual: 10:30
2. Calcula número siguiente en PRD
3. Actualiza PRD_DOCUMENTS/PRD_20260225.md:
   ### ✅ N. Revisar correos — **10:30**
   **Descripción**  
   Revisión diaria de correos...
   **Solución**  
   Se procesaron 23 correos nuevos...
4. Confirma actualización en PRD
```

**Actualización en:**
```
PRD_DOCUMENTS/PRD_20260225.md  ← Se actualiza con tarea
```

### Generar Reporte de Horas

```
Usuario: "Genera el reporte de horas de hoy"
Claude:
1. Ejecuta: python scripts/generate_hours_report.py PRD_20260225.md
2. Lee PRD_DOCUMENTS/PRD_20260225.md
3. Extrae tareas y timestamps
4. Calcula duración entre tareas
5. Genera REPORTS/HORAS_PRD_20260225.md
6. Confirma generación:
   ✅ Reporte guardado: ~/Documents/prd_diarios/REPORTS/HORAS_PRD_20260225.md
   📊 Horas trabajadas: 7h 30m
   📋 Tareas registradas: 5
```

**Output:**
```
REPORTS/HORAS_PRD_20260225.md
  - Desglose por tarea
  - Duración de cada una
  - Horas totales
  - Promedio
```

### Generar Resumen del Día

```
Usuario: "Dame un resumen del día" o "Genera resumen del día"
Claude:
1. Ejecuta: python scripts/generate_day_summary.py
2. Analiza DAILY_WORK/260225/
3. Lee metadatos de TODOS los archivos
4. Determina hora de inicio (primer archivo creado)
5. Extrae tareas del PRD
6. Calcula horas trabajadas
7. Genera REPORTS/RESUMEN_260225.md con:
   - Información general (hora inicio, tareas, horas, documentos)
   - Tareas realizadas con timestamps
   - Tareas pendientes
   - Lista de documentos con metadatos
8. Confirma:
   ✅ Resumen guardado: ~/Documents/prd_diarios/REPORTS/RESUMEN_260225.md
   📊 Estadísticas:
      - Tareas completadas: 5
      - Tareas pendientes: 2
      - Horas trabajadas: 7h 30m
      - Documentos: 8
```

**Output:**
```
REPORTS/RESUMEN_260225.md
  - Información del día
  - Tareas completadas/pendientes
  - Documentos generados
  - Análisis de metadatos
```

## Características Clave

✅ **Carpetas Diarias** - Organiza todo en carpetas YYMMDD (260225, 260226...)  
✅ **Gestión Unificada** - PRD + conversaciones + documentos en un solo lugar  
✅ **Metadatos de Archivos** - Rastrea fecha creación/modificación de documentos  
✅ **Hora de Inicio Automática** - Detecta cuándo empezó el día (primer archivo)  
✅ **Formato Jerárquico** - Estructura clara con encabezados H3 y emojis  
✅ **Timestamps Exactos** - Registra hora de inicio de cada tarea  
✅ **Documentación Completa** - Descripción + Solución para auditoría  
✅ **Resumen Automático del Día** - Analiza carpeta y genera reporte completo  
✅ **Reportes de Horas** - Scripts Python/PowerShell generan horas trabajadas  
✅ **Gestión de Pendientes** - Seguimiento de tareas en progreso  
✅ **Git-friendly** - Markdown puro, fácil de versionear  
✅ **Rastreabilidad Completa** - Auditoría diaria con toda la información

## Ejemplos de Registros Reales

### Tarea Simple (Bug Fix)

```markdown
### ✅ 2. Corregir validación en formulario — **11:15**

**Descripción**  
Usuario reportó error en validación de email en formulario de contacto. La validación rechazaba emails válidos con subdominios. Impacta signup de nuevos usuarios.

**Solución**  
Se identificó regex incorrecto en campo email (patrón muy restrictivo). Se actualizó patrón de validación a RFC 5322. Se testeó con 50 casos de prueba. Desplegado en producción. Validado con clientes específicos.
```

### Tarea Compleja (Integración)

```markdown
### ✅ 5. Integración con API de Stripe — **14:45**

**Descripción**  
Cliente solicita añadir nuevo proveedor de pagos (Stripe) al sistema. Requerido conectar a sistema actual, modificar flujo de checkout y actualizar documentación. Esta es actividad crítica para Q1.

**Solución**  
Se implementó cliente Stripe official. Se integraron webhooks para confirmación de pago y reembolsos. Se actualizó checkout para soportar múltiples proveedores (Stripe + PayPal). Testing completado con casos de éxito y error. Demo realizado con cliente. Documentación actualizada.
```

## Scripts Disponibles

### setup_config.py (NUEVO - IMPORTANTE!)

Asistente interactivo para configurar las rutas de trabajo.

```bash
python scripts/setup_config.py              # Ejecuta wizard interactivo
python scripts/setup_config.py --show       # Muestra configuración actual
python scripts/setup_config.py --reset      # Resetea la configuración
```

**Características:**
- Interfaz interactiva y amigable
- Configura PRD_DOCUMENTS, DAILY_WORK, REPORTS, ARCHIVES
- Valida rutas y crea directorios
- Guarda configuración en config.json
- Compatible con macOS, Linux y Windows

### create_daily_folder.py

### create_daily_folder.py

Crea una carpeta diaria con formato YYMMDD en la carpeta DAILY_WORK.

```bash
python scripts/create_daily_folder.py [--date 20260225] [--path ./custom]
```

**Características:**
- Crea carpeta en DAILY_WORK (configurado en setup_config.py)
- Formato YYMMDD (260225 para 25 de febrero de 2026)
- Genera README.md dentro con información del día
- Verifica si la carpeta ya existe antes de crear

**Estructura creada:**
```
DAILY_WORK/260225/
  ├── README.md
  └── (vacía, lista para documentos)
```

### create_daily_prd.py (ACTUALIZADO)

Crea un nuevo archivo PRD_YYYYMMDD.md en la carpeta PRD_DOCUMENTS.

```bash
python scripts/create_daily_prd.py [--date 20260216] [--path ./custom]
```

**Características:**
- Crea PRD en carpeta PRD_DOCUMENTS (configurada en setup_config.py)
- Nombre: PRD_20260225.md
- Incluye estructura inicial y timestamp
- También usable para PRDs de otros días/proyectos

**Estructura creada:**
```
PRD_DOCUMENTS/
  ├── PRD_20260225.md
  ├── PRD_20260226.md
  └── PRD_20260227.md
```

### create_daily_prd.ps1

Versión PowerShell de creación de PRD con soporte para carpetas diarias.

```powershell
.\scripts\create_daily_prd.ps1 -Date "2026-02-16" -Output "./path"
```

### generate_day_summary.py (ACTUALIZADO)

Analiza todos los archivos de la carpeta diaria (DAILY_WORK/YYMMDD/) y genera un resumen completo en REPORTS/.

```bash
python scripts/generate_day_summary.py [--date 20260225] [--path ./custom] [--output ./custom]
```

**Características:**
- Lee **metadatos** de todos los archivos en DAILY_WORK/YYMMDD/
- Determina **hora de inicio del día** (primer archivo creado)
- Extrae **tareas del PRD** (completadas y pendientes)
- Calcula **horas trabajadas** basado en timestamps
- Lista **todos los documentos** generados en el día
- Guarda **RESUMEN_YYMMDD.md** en carpeta REPORTS

**Input:** `DAILY_WORK/260225/` (carpeta con documentos del día)  
**Output:** `REPORTS/RESUMEN_260225.md`

### generate_hours_report.py (ACTUALIZADO)

Analiza un PRD diario y genera reporte detallado de horas trabajadas en carpeta REPORTS/.

```bash
python scripts/generate_hours_report.py PRD_20260216.md [--output ./custom]
```

**Características:**
- Lee el PRD de PRD_DOCUMENTS/
- Extrae timestamps de tareas
- Calcula duración entre tareas
- Guarda reporte en carpeta REPORTS
- Genera desglose detallado de horas

**Input:** `PRD_DOCUMENTS/PRD_20260225.md`  
**Output:** `REPORTS/HORAS_PRD_20260225.md`

### generate_hours_report.ps1

Versión PowerShell de generación de reportes.

```powershell
.\scripts\generate_hours_report.ps1 -PRDFile "PRD_260216.md" -Output "./reports"
```

## Checklist de Completitud

**Configuración Inicial (Primera vez):**
- [ ] Ejecutaste `python scripts/setup_config.py`
- [ ] Configuraste correctamente PRD_DOCUMENTS
- [ ] Configuraste correctamente DAILY_WORK
- [ ] Configuraste correctamente REPORTS
- [ ] Las rutas se crearon correctamente

**Cada Día:**
- [ ] Creaste carpeta diaria con `create_daily_folder.py`
- [ ] Creaste PRD con `create_daily_prd.py`
- [ ] Registraste todas las tareas con descripción clara
- [ ] Cada solución explica QUÉ se hizo y POR QUÉ
- [ ] Hay timestamps para cada tarea
- [ ] Tareas pendientes están documentadas
- [ ] Guardaste todos los documentos en DAILY_WORK/YYMMDD/

**Fin de Día:**
- [ ] Todos los documentos están en DAILY_WORK/260225/
- [ ] PRD está actualizado en PRD_DOCUMENTS/
- [ ] Generaste el resumen: `generate_day_summary.py`
- [ ] Verificaste que RESUMEN_260225.md está en REPORTS/
- [ ] Generaste el reporte de horas (opcional)
- [ ] Validaste que los totales de horas son correctos

## Referencias

- [Estructura Detallada](references/structure.md) - Detalles técnicos completos
- [Plantilla](assets/template.md) - Plantilla lista para usar
- [Skill PRD](../prd/SKILL.md) - Para análisis profesionales profundos

## Licencia

MIT License - Libre para usar y modificar

---
> Source: [lunasoft2001/github-copilot-skill-prddiario](https://github.com/lunasoft2001/github-copilot-skill-prddiario) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
