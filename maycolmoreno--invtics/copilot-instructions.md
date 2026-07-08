## invtics

> CRESIO es un sistema de gestión de activos TIC, inventario operativo, compras, recepción, custodias, mantenimientos, bodegas, stock y trazabilidad.

# AGENTS.md — Reglas de trabajo para CRESIO

## Contexto del proyecto

CRESIO es un sistema de gestión de activos TIC, inventario operativo, compras, recepción, custodias, mantenimientos, bodegas, stock y trazabilidad.

El proyecto está dividido en dos capas principales:

```text
Browser
  ↓
consumogestionactivosapi
BFF MVC / Thymeleaf / puerto 8081
  ↓ RestClient
gestionactivosapi
API Backend / puerto 8084
  ↓
PostgreSQL
```

`consumogestionactivosapi` es un BFF MVC/Thymeleaf.
No es el backend principal de negocio.
La persistencia, reglas profundas, entidades JPA, migraciones Flyway y lógica transaccional viven principalmente en `gestionactivosapi`.

## Stack

### BFF Web

* Java 17.
* Spring Boot.
* Spring MVC.
* Thymeleaf.
* Bootstrap 5.
* RestClient hacia API backend.
* Frontend Enterprise UI con clases `cui-*`.

### Backend API

* Java 17.
* Spring Boot.
* PostgreSQL.
* JPA.
* Flyway.
* Servicios de dominio/aplicación.
* Reglas transaccionales.

## Arquitectura funcional objetivo

CRESIO debe operar como plataforma ITAM empresarial, no como sistema de formularios aislados.

Regla UX principal:

```text
Datos visibles → Selección → Acción contextual → Drawer → Confirmación
```

No usar formularios permanentes si la acción puede nacer desde una fila.

## Reglas obligatorias generales

No modificar base de datos salvo aprobación explícita.

No modificar entidades, DTOs, servicios, controladores, endpoints o migraciones sin aprobación.

No crear endpoints nuevos sin aprobación.

No cambiar mappings existentes sin aprobación.

No cambiar nombres de inputs de formularios sin verificar DTOs BFF y backend.

No romper rutas antiguas.

No tocar Flutter salvo solicitud explícita.

No tocar API backend salvo que se indique explícitamente.

No mezclar cambios visuales con cambios de dominio.

No implementar múltiples fases en una sola respuesta.

No agregar funcionalidades especulativas.

No usar datos inventados para llenar vacíos del modelo.

## Flujo de trabajo obligatorio

Antes de modificar archivos:

1. Analizar código relacionado.
2. Identificar capa afectada:

   * BFF MVC/Thymeleaf.
   * API backend.
   * BD/Flyway.
   * UI estática.
3. Explicar el problema.
4. Proponer plan.
5. Listar archivos a crear/modificar.
6. Indicar riesgos.
7. Esperar aprobación.

Después de modificar:

1. Mostrar archivos creados/modificados.
2. Explicar cambios.
3. Ejecutar:

   * `.\mvnw.cmd compile`
   * `.\mvnw.cmd test`
   * `git diff --check`
4. Reportar resultado de validaciones.
5. Reportar riesgos.
6. No continuar con la siguiente fase sin aprobación.

## Reglas de Compras y Recepción

Regla central:

```text
Compra define lo solicitado.
Recepción convierte lo recibido en activo o stock.
```

Flujo objetivo:

```text
OrdenCompra
  ↓
OrdenCompraDetalle
  ↓
RecepcionLote
  ↓
Equipo / StockConsumibleBodega
  ↓
MovimientoInventario
  ↓
Custodia / Traslado / Mantenimiento / Baja
```

No debe existir recepción libre desconectada de una OC.

No usar pantallas independientes como solución principal:

```text
Recepción de activo
Recepción de consumible
```

La recepción debe realizarse desde:

```text
Gestionar OC → Línea de detalle → Recibir
```

Cada línea de OC debe mostrar:

* Descripción.
* Tipo: `ACTIVO` o `STOCK`.
* Cantidad solicitada.
* Cantidad recibida.
* Cantidad pendiente.
* Estado.
* Acción contextual `Recibir`.

Si el detalle es `ACTIVO`:

* Genera activos individuales.
* Requiere serial.
* Debe asociarse a OC, detalle y lote de recepción.
* Debe generar código CRESIO.
* Debe quedar vinculado a bodega.
* Debe guardar condición al recibir.
* Puede registrar datos técnicos opcionales.

Si el detalle es `STOCK`:

* No genera código individual.
* Incrementa stock en bodega.
* Debe asociarse a OC, detalle y lote de recepción.
* Debe generar movimiento de inventario.

## Estados de Orden de Compra

Estados válidos:

```text
BORRADOR
EMITIDA
RECEPCION_PARCIAL
RECIBIDA
CANCELADA
```

Reglas:

* `BORRADOR` permite edición.
* `EMITIDA` permite recepción.
* `RECEPCION_PARCIAL` permite recepción pendiente.
* `RECIBIDA` no permite nuevas recepciones.
* `CANCELADA` no permite operaciones.

Si en el sistema legacy existe `RECIBIDA_PARCIAL`, tratarlo como valor de compatibilidad temporal. No introducirlo en nuevas vistas como estado principal.

## Estados de OrdenCompraDetalle

Estados válidos:

```text
PENDIENTE
PARCIAL
COMPLETO
CANCELADO
```

Reglas:

* `cantidadRecibida = 0` → `PENDIENTE`.
* `0 < cantidadRecibida < cantidadSolicitada` → `PARCIAL`.
* `cantidadRecibida = cantidadSolicitada` → `COMPLETO`.
* No permitir `cantidadRecibida > cantidadSolicitada`.

## RecepcionLote

Todo lote debe tener:

* `uuid`.
* OC asociada.
* Detalle OC asociado.
* Cantidad recibida.
* Estado.
* Bodega destino.
* `recepcionadoPor`.
* `recepcionadoEn`.
* Observación opcional.

No usar `id_custodio_receptor` como responsable de recepción logística en nuevas fases. La recepción la ejecuta un usuario/custodio de bodega, pero la custodia del activo ocurre después.

## Reglas de bodegas

Una bodega debe tener custodio responsable.

El custodio responsable de bodega debe:

* Estar activo.
* Tener cargo/departamento asociado.
* Pertenecer al departamento `TECNOLOGÍAS E INNOVACIÓN`.

Validar esta regla en backend/API, no solo en Thymeleaf.

La comparación del departamento debe ser robusta:

* Ignorar mayúsculas/minúsculas.
* Evitar fallos por tildes cuando sea posible.

## Reglas de activos

Estados válidos del ciclo de vida del activo:

```text
EN_BODEGA
ASIGNADO
EN_REPARACION
EN_TRANSITO
DADO_DE_BAJA
```

Reglas de asignación:

Un activo solo puede asignarse si:

* `estadoInventario = EN_BODEGA`.
* `etiquetado = true`.
* `bodegaActual != null`.
* No tiene custodia activa.
* Custodio destino está activo.

No asignar activos:

* `ASIGNADO`.
* `EN_REPARACION`.
* `EN_TRANSITO`.
* `DADO_DE_BAJA`.

Si el activo no está etiquetado, no puede salir de bodega.

## Reglas de Stock

La pantalla Stock debe ser una consola de existencias.

No usar paneles permanentes para entregar o trasladar.

Flujo correcto:

```text
Stock → Fila de consumible/bodega → Entregar / Trasladar / Movimientos → Drawer
```

La fila debe aportar el contexto:

* Consumible.
* Bodega.
* Cantidad disponible.
* Estado.

No volver a seleccionar consumible ni bodega si la acción nace desde una fila.

## Reglas de Traslados

La pantalla Traslados debe ser principalmente:

* Historial.
* Consulta.
* Control de movimientos.

No debe tener dos formularios permanentes:

```text
Traslado de activo
Traslado de consumible
```

Los traslados deben iniciar preferentemente desde:

* Stock, si es `STOCK`.
* Listado/expediente de activo, si es `ACTIVO`.

Si existe botón global `Nuevo traslado`, debe abrir drawer/wizard, no formularios fijos paralelos.

## Reglas UX Enterprise

Evitar:

* Formularios gigantes permanentes.
* KPIs decorativos.
* Summary strips que repiten datos de la tabla.
* Botones deshabilitados sin explicación.
* Duplicidad de acciones.
* Pantallas que obligan a seleccionar datos ya visibles.

Aplicar:

* Tablas full-width.
* Acciones contextuales por fila.
* Drawers con datos precargados.
* Estados con badges.
* Progreso por línea cuando aplique.
* Empty states útiles.
* Confirmación para acciones peligrosas.
* CSS con clases `cui-*`.
* Sin estilos inline.
* Sin `onclick`/`onsubmit` inline si se puede usar JS progresivo.

## Reglas sobre KPIs y Summary Strips

Regla obligatoria:

```text
Si el usuario puede ver, filtrar o contar el dato desde la tabla,
no debe mostrarse como KPI ni Summary Strip.
```

Solo mostrar arriba:

* Riesgos.
* Pendientes.
* Excepciones.
* Alertas accionables.
* Anomalías operativas.

No mostrar:

* Totales simples.
* Activos/Inactivos.
* Cerradas/Activas si ya existe columna Estado.
* Categorías que ya existen como filtros.
* Conteos de tabla.

## Reglas de integración de empleados/custodios

Si se consume información de empleados desde JSON externo:

No consultar el JSON en vivo durante una asignación.

Usar sincronización controlada:

```text
JSON empleados
  ↓
Servicio de sincronización
  ↓
Tabla local empleados/custodios
  ↓
CRESIO valida asignaciones localmente
```

Reglas:

* No asignar activos o gastos a empleados inactivos.
* Detectar empleados fuera de servicio.
* Generar alerta si un empleado inactivo tiene activos asignados.
* Registrar fecha de última sincronización.
* Registrar cambios relevantes.
* No guardar credenciales en texto plano.

## Reglas de BD/Flyway

No ejecutar `DROP TABLE` sin auditoría previa.

Antes de eliminar tablas:

1. Verificar si tienen datos.
2. Verificar si existen FKs.
3. Verificar entidades JPA.
4. Verificar repositorios.
5. Verificar controladores/servicios.
6. Verificar vistas que dependan de ellas.
7. Proponer migración de datos.
8. Esperar aprobación.

Tablas sospechosas actuales que requieren auditoría antes de eliminar:

* `activos`.
* `actualizacion_activos`.
* `empresas`.
* `checklist_categoria`.

No eliminar sin revisión.

## Prioridades actuales

1. Corregir drift técnico BD/JPA de compras:

   * `ordenes_compra.version`.
   * Check de estados de OC.
   * Check de tipo item en detalle.
2. Consolidar flujo:

   * OC → Detalle → Recepción por línea.
3. Eliminar recepción libre como flujo principal.
4. Rediseñar Stock:

   * acción contextual por fila.
   * drawers precargados.
5. Rediseñar Traslados:

   * historial y acciones desde Stock/Activos.
6. Validar bodegas con custodio de `TECNOLOGÍAS E INNOVACIÓN`.
7. Completar reglas de ciclo de vida del activo.
8. Mantener UI Enterprise limpia y sin formularios redundantes.
9. Tests básicos por cada cambio funcional.
10. Resiliencia BFF:

* timeouts.
* ControllerAdvice.
* sesiones.
* circuit breaker si se aprueba.

---
> Source: [maycolmoreno/InvTICS](https://github.com/maycolmoreno/InvTICS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
