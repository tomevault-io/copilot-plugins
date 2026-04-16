## studio

> Responde siempre en español y recuerda que estamos trabajando en windows

Responde siempre en español y recuerda que estamos trabajando en windows
# Contexto y Planes de Gemini

## Objetivo: Aplicación de Escritorio con Tauri + Node.js

El objetivo es crear una aplicación de escritorio nativa usando Tauri como lanzador y un backend de Node.js (Express.js) funcionando como un proceso "Sidecar". El backend manejará toda la lógica de negocio y la comunicación con la base de datos SQLite. El frontend será la interfaz de usuario existente construida con Next.js/React.

### Arquitectura Final:

1.  **Tauri (Rust):** Actúa como el envoltorio nativo de la aplicación. Su principal responsabilidad es crear la ventana y lanzar/gestionar el proceso del servidor de Node.js.
2.  **Backend (Node.js):** Un servidor Express.js que se ejecuta localmente. Este servidor se conecta a la base de datos SQLite y expone una API REST local (ej. `http://localhost:3001/api/...`) para que el frontend la consuma.
3.  **Frontend (React):** La interfaz de usuario existente. Realizará llamadas `fetch` a la API del backend de Node.js para obtener y enviar datos.

## Plan de Implementación

1.  **Configurar el Entorno:** Preparar la estructura de carpetas (`src-backend`), instalar dependencias de Node.js (`express`, `sqlite3`) y configurar Tauri para permitir la ejecución de procesos sidecar.
2.  **Codificar el Backend:** Desarrollar la API en Node.js con endpoints para todas las operaciones CRUD (Productos, Compras, Ventas) y la generación de reportes.
3.  **Conectar el Frontend:** Modificar los componentes de React para que se comuniquen con la API del backend local en lugar de usar datos estáticos.
4.  **Implementar el Lanzador:** Escribir el código en `main.rs` (Rust) para iniciar y detener el servidor de Node.js junto con la aplicación.

## Plan de Base de Datos (SQLite)

El archivo `database.sql` define el esquema. El backend de Node.js será el único que interactúe con esta base de datos.

---

## Endpoints de la API Requeridos

A continuación se detallan los endpoints necesarios para dar vida al frontend.

### Productos (`/api/products`)

*   **`GET /api/products`**: Obtener la lista completa de productos.
*   **`POST /api/products`**: Crear un nuevo producto.
*   **`GET /api/products/:id`**: Obtener los detalles de un solo producto.
*   **`PUT /api/products/:id`**: Actualizar un producto existente.
*   **`DELETE /api/products/:id`**: Eliminar un producto.

### Movimientos de Inventario (`/api/inventory`)

*   **`POST /api/inventory/movements`**: Registrar un nuevo movimiento (compra, venta, etc.).

### Reportes (`/api/reports`)

*   **`POST /api/reports/inventory`**: Generar un reporte de inventario.
*   **`POST /api/reports/sales`**: Generar un "Libro de Venta".
*   **`POST /api/reports/purchases`**: Generar un "Libro de Compra".
*   **`GET /api/reports`**: Obtener historial de reportes generados.

### Dashboard (`/api/dashboard`)

> **Nota:** Los endpoints del Dashboard (`/summary` y `/recent-sales`) actualmente usan valores de marcador de posición (placeholders) para datos como los porcentajes de cambio y la información del cliente. Estos deben ser reemplazados por consultas dinámicas a la base de datos cuando el esquema lo permita.

*   **`GET /api/dashboard/summary`**: Obtener estadísticas clave para el panel.
*   **`GET /api/dashboard/recent-sales`**: Obtener lista de ventas recientes.

### Configuración (`/api/settings`)

*   **`GET /api/settings/store`**: Obtener los detalles de la tienda.
*   **`PUT /api/settings/store`**: Actualizar los detalles de la tienda.
*   **`POST /api/database/backup`**: Iniciar respaldo de la base de datos.
*   **`POST /api/database/restore`**: Iniciar restauración de la base de datos.

### Health Check (`/api/health`)

*   **`GET /api/health`**: Verifica que el backend esté funcionando correctamente. Devuelve un estado `ok`.

---

## Estado Actual del Proyecto

La estructura inicial del proyecto ya ha sido creada, siguiendo la arquitectura definida en este documento. Existen los archivos base para el backend de Node.js (`src-backend`), el frontend de Next.js (`src`) y el lanzador de Tauri (`src-tauri`).

El proyecto se encuentra en la **fase de desarrollo e implementación activa**. El trabajo futuro debe centrarse en desarrollar los endpoints de la API y conectar los componentes del frontend, en lugar de volver a generar el código base.

**Última acción:** Se implementó un sistema de "pre-arranque" o pantalla de carga. El frontend ahora sondea el nuevo endpoint `/api/health` en el backend y muestra una pantalla de espera. La interfaz de usuario principal no se renderizará hasta que el backend confirme que está completamente operativo, evitando así errores de renderizado y mejorando la experiencia de inicio de la aplicación.

---

## Últimas Actualizaciones (9 de Julio de 2025)

Se ha realizado una revisión y refactorización exhaustiva de varios endpoints del backend para asegurar la consistencia de los datos y prevenir errores en el frontend.

1.  **Alineación de Datos (Producto):**
    *   Se modificaron los endpoints `GET /api/products` y `GET /api/products/:id`.
    *   Ahora devuelven los campos `stock` y `price` (usando alias de SQL) en lugar de `current_stock` y `average_cost`, que es lo que el frontend esperaba. Esto solucionó un error crítico que impedía renderizar la página de productos.

2.  **Robustecimiento del Backend:**
    *   Se revisaron los endpoints del Dashboard (`/api/dashboard/summary` y `/api/dashboard/recent-sales`) para que el backend maneje los valores nulos o indefinidos.
    *   Los cálculos y los datos de marcador de posición (placeholders) ahora se resuelven en el servidor, enviando una estructura de datos limpia y predecible al cliente.

3.  **Nuevos Endpoints (Compras y Ventas):**
    *   Se crearon los endpoints `GET /api/purchases` y `GET /api/sales`.
    *   Estos endpoints proporcionan listas dedicadas para el historial de compras (`ENTRADA`) y ventas (`SALIDA`), filtrando la tabla `inventory_movements`. Esto permite que las páginas de Compras y Ventas del frontend funcionen correctamente.

4.  **Implementación del Estado de Productos y Corrección de Errores:**
    *   Se solucionó un error crítico (`SQLITE_ERROR: no such column: status`) que impedía la carga de las páginas de productos, compras y ventas.
    *   Se añadió una columna `status` a la tabla `products` en la base de datos (`schema.sql`) para gestionar si un producto está 'Activo' o 'Inactivo'.
    *   Se actualizaron los endpoints del backend (`POST`, `PUT`, `GET` para productos) para que sean compatibles con el nuevo campo `status`.
    *   Se refactorizó por completo la página de productos del frontend (`products/page.tsx`) para implementar la funcionalidad de **edición**, que no existía.
    *   El diálogo de productos ahora sirve tanto para crear como para editar, y utiliza un componente `Switch` para cambiar el estado del producto.
    *   Se actualizaron las definiciones de tipos (`Product`) y las llamadas a la API (`createProduct`, `updateProduct`) para alinearlas con los cambios.
---

## Decisiones de Arquitectura y Lecciones Aprendidas

### Manejo de Operaciones de Lote (Batch Operations)

**Fecha:** 10 de Julio de 2025

**Lección:** Se detectó un error crítico al procesar operaciones que involucran múltiples registros, como una compra con varios productos.

*   **El Problema:** Al registrar una compra, el frontend enviaba múltiples peticiones en paralelo al backend (una por cada producto). El backend intentaba abrir una transacción de base de datos (`BEGIN TRANSACTION`) para cada petición, resultando en el error `SQLITE_ERROR: cannot start a transaction within a transaction` porque SQLite no permite transacciones anidadas o concurrentes de esta manera.

*   **La Solución:** Se refactorizó el backend para manejar la operación completa como un lote atómico.
    1.  Se creó un nuevo endpoint de lote: `POST /api/purchases`.
    2.  Este endpoint acepta un array con todos los productos de la compra en **una sola llamada a la API**.
    3.  El método del backend envuelve todo el proceso (recorrer los productos, actualizar el stock de cada uno, registrar los movimientos) dentro de una **única transacción** (`BEGIN`...`COMMIT`). Si un solo producto falla, se revierten todos los cambios (`ROLLBACK`), garantizando la integridad de los datos.

*   **Acción Requerida para la Página de Ventas:** Se debe replicar la funcionalidad completa de la sección de Compras en la sección de Ventas, incluyendo:
    1.  **Operaciones de Lote:** Crear un endpoint `POST /api/sales` que procese la venta completa en una única transacción para evitar errores de concurrencia.
    2.  **Historial y Recibos:** Implementar un modal de "Historial de Ventas" que permita visualizar ventas pasadas y reimprimir sus respectivos recibos.
    3.  **Edición Segura:** Añadir la capacidad de editar ventas desde el historial. El backend (`PUT /api/sales`) debe seguir el patrón de **anulación y re-creación** para garantizar la integridad del inventario y mantener un rastro de auditoría.
    4.  **Interfaz de Usuario Pulida:** Reemplazar los botones de texto en los modales de historial y recibo por **botones de icono con tooltips** (para acciones como Editar, Ver Recibo, Imprimir, etc.), manteniendo la consistencia visual con la sección de Compras.

---
## Tareas Pendientes para la Página de Ventas (Replicar Funcionalidad de Compras)

**Fecha:** 11 de Julio de 2025

La sección de **Compras** ha sido refactorizada y mejorada significativamente. La sección de **Ventas** debe ser actualizada para incorporar las mismas funcionalidades y patrones de diseño, garantizando una experiencia de usuario consistente y una lógica de negocio robusta.

### 1. Lógica de Backend y API

*   **Endpoint de Lote (`POST /api/sales`):** Crear un endpoint que acepte un array de productos para procesar la venta completa en una única transacción atómica.
*   **Endpoint de Edición (`PUT /api/sales`):** Implementar la edición de ventas. Debe seguir el patrón de **anulación y re-creación**, marcando los movimientos originales con `status = 'Reemplazado'`.
*   **Endpoint de Anulación (`DELETE /api/sales`):** Crear un endpoint que reciba los IDs de los movimientos de una venta y los marque como `status = 'Anulado'`, revirtiendo su impacto en el inventario.

### 2. Interfaz de Usuario (Página de Ventas)

*   **Formulario de Venta:**
    *   Implementar la consolidación de productos en el frontend *antes* de enviar los datos.
    *   Añadir un diálogo de confirmación (`SalesConfirmationDialog`) que muestre la lista de productos consolidada antes de registrar la venta.
    *   Permitir la creación de nuevos productos directamente desde el formulario de ventas, asegurando que el nuevo producto se añada correctamente a la línea de venta actual.

### 3. Historial de Ventas

*   **Crear `SalesHistoryDialog.tsx`:**
    *   Debe mostrar una lista de todas las ventas (activas y anuladas).
    *   **Buscador:** Incluir un campo de búsqueda único para filtrar por cliente o número de factura/recibo.
    *   **Diferenciación Visual:** Las ventas anuladas deben aparecer visualmente distintas (atenuadas, tachadas) y con una `Badge` de "Anulada".
    *   **Acciones por Fila:**
        *   **Ver Recibo:** Botón para abrir un diálogo con los detalles de la venta.
        *   **Editar:** Botón (habilitado solo para ventas activas) para cargar los datos de la venta en el formulario principal.
        *   **Anular:** Botón (habilitado solo para ventas activas) que, tras confirmación, llame al endpoint `DELETE /api/sales`.

### 4. Kardex de Producto

*   **Integrar Vista de Detalle:** Desde la página de ventas, al seleccionar un producto, se debería poder acceder a su kardex (`ProductDetailDialog`) para ver su historial de movimientos, incluyendo las ventas.
*   **Registrar Salidas:** Habilitar la opción de registrar "Retiros" y "Autoconsumo" desde el kardex, tal como se hizo en la sección de productos.

### 5. Notificaciones

*   Utilizar el sistema de notificaciones centralizado (`use-toast.tsx`) para todos los mensajes de éxito y error, asegurando consistencia visual y de comportamiento (temporizador de 5 segundos, iconos, etc.).
---
## Tarea Actual: Reporte de Inventario en Excel (Formato Fiscal)

**Fecha:** 13 de Julio de 2025

**Objetivo:** Implementar la capacidad de exportar el reporte de inventario a un archivo de Excel (`.xlsx`) con un formato específico que cumple con normativas fiscales, incluyendo encabezados de la empresa, estructura de tabla detallada y valores calculados.

### Plan de Implementación por Fases

1.  **Fase 1: Habilitar la Interfaz de Usuario (Frontend)**
    *   Añadir un botón "Exportar a Excel" en la página de reportes.
    *   Crear el estado de carga y las funciones controladoras (`handler`) necesarias para la nueva acción.
    *   Implementar la función de llamada a la API en `lib/api.ts`.

2.  **Fase 2: Crear Endpoint de Marcador de Posición (Backend)**
    *   Instalar la dependencia `exceljs` en `src-backend`.
    *   Crear el endpoint `POST /api/reports/inventory-excel`.
    *   Inicialmente, este endpoint solo registrará en consola las fechas recibidas y devolverá un mensaje de éxito (`200 OK`) para verificar la conexión.

3.  **Fase 3: Generación del Archivo Excel con Formato y Datos Estáticos (Backend)**
    *   Modificar el endpoint para que utilice `exceljs`.
    *   Construir la estructura del archivo Excel:
        *   Añadir y fusionar celdas para el encabezado (Nombre de la empresa, RIF, Título del reporte).
        *   Crear la fila de cabeceras de la tabla.
        *   Aplicar estilos básicos (negrita, alineación).
    *   Poblar la tabla con **datos estáticos (dummy data)** para simular el resultado final.
    *   Configurar las cabeceras HTTP de la respuesta para que el navegador descargue el archivo `.xlsx` generado.

4.  **Fase 4: Integración de Datos Reales (Backend)**
    *   Reemplazar los datos estáticos con consultas reales a la base de datos.
    *   Obtener los detalles de la tienda desde la tabla `store_settings`.
    *   Para cada producto, calcular los valores requeridos (existencia anterior, entradas, salidas, etc.) consultando la tabla `inventory_movements` dentro del rango de fechas especificado.
    *   Poblar el archivo Excel con los datos dinámicos obtenidos.
---
## Anexo: Lógica de Valoración de Inventario (Costo Promedio Ponderado)

### 1. ¿Cómo funciona el Costo Promedio Ponderado?

Este método recalcula el costo unitario promedio de un artículo cada vez que se realiza una nueva compra (una entrada al inventario). La fórmula es:

$$
\text{Nuevo Costo Promedio} = \frac{(\text{Unidades Actuales} \times \text{Costo Promedio Actual}) + (\text{Unidades de la Nueva Compra} \times \text{Costo de la Nueva Compra})}{(\text{Unidades Actuales} + \text{Unidades de la Nueva Compra})}
$$

**Ejemplo práctico:**

1.  **Inventario Inicial:** Tienes 10 unidades a un costo de 100 Bs. cada una.
    * *Valor Total:* 1,000 Bs.
    * *Costo Promedio:* 100 Bs.
2.  **Haces una Compra:** Adquieres 15 unidades más, pero esta vez a 120 Bs. cada una.
    * *Valor de la Compra:* 1,800 Bs.
3.  **Cálculo del Nuevo Promedio:**
    * *Nuevo Valor Total:* 1,000 Bs. (inicial) + 1,800 Bs. (compra) = 2,800 Bs.
    * *Nuevas Unidades Totales:* 10 (inicial) + 15 (compra) = 25 unidades.
    * **Nuevo Costo Promedio Ponderado:** 2,800 Bs. / 25 unidades = **112 Bs. por unidad**.

A partir de este momento, **todas las salidas** de inventario (ventas, retiros) se valorarán a este nuevo costo de 112 Bs., hasta que una nueva compra obligue a recalcularlo.

### 2. ¿Qué valor (Anterior, Actual, Promedio) se usa para cada columna?

Esta es la parte crucial para llenar el reporte correctamente. Cada columna de la sección de valores monetarios se calcula usando un costo unitario específico. Igual que las salidas por ventas, se valoran utilizando el **`VALOR PROMEDIO UNITARIO EN BS`**. Son egresos de inventario y deben registrarse a su costo.
    * **Fórmula:** `Retiros (Unidades)` x `Valor Promedio Unitario (Bs)`.

* **Existencia Actual (en Valor):**
    * Este es el resultado final. Se puede obtener de dos maneras que deben coincidir:
        1.  **Por cálculo aritmético:** `Existencia Anterior (Valor) + Entradas (Valor) - Salidas (Valor) - Retiros (Valor) - Auto-consumo (Valor)`.
        2.  **Por valoración final:** `Existencia Actual (Unidades)` x `Valor Promedio Unitario ACTUAL (Bs)`. El `Valor Promedio Actual` será el último costo promedio que se calculó en el período.

En resumen, la clave es: **las salidas de inventario, sin importar su motivo (venta, retiro, etc.), siempre se valoran al costo promedio ponderado que esté vigente en ese preciso momento.** Este es el método que la administración tributaria venezolana (SENIAT) verifica para validar el costo de ventas declarado.
---
## Tarea Actual: Refactorización de Transacciones

**Fecha:** 13 de Julio de 2025

**Objetivo:** Modificar la base de datos y el backend para utilizar un identificador de transacción único e inmutable (`transaction_id`) y campos estructurados para la fecha y los datos de la entidad (cliente/proveedor), eliminando la dependencia del campo `description` para almacenar esta información crítica.

### Plan de Implementación

1.  **Paso 1: Modificar la Base de Datos (Backend)**
    *   **[COMPLETADO]** Actualizar `src-backend/schema.sql` para añadir las columnas `transaction_id`, `transaction_date`, `entity_name`, y `entity_document` a la tabla `inventory_movements`.
    *   **[COMPLETADO]** Crear un script de migración (`src-backend/temp_alter.sql`) para aplicar estos cambios a la base de datos existente.

2.  **Paso 2: Refactorizar la Capa de API (`src/lib/api.ts`)**
    *   **[COMPLETADO]** Modificar las funciones de creación (ej. `createPurchase`) para que dejen de empaquetar datos en `description` y los envíen como campos separados.
    *   **[COMPLETADO]** Actualizar las funciones de lectura (ej. `getPurchaseHistory`) para que usen `transaction_id` para agrupar y solicitar recibos.

3.  **Paso 3: Refactorizar el Servidor Backend (`src-backend/index.js`)**
    *   **[COMPLETADO]** Actualizar los endpoints `POST` para que reciban los nuevos campos estructurados, generen un `transaction_id` único por lote y lo guarden en cada movimiento.
    *   **[COMPLETADO]** Actualizar los endpoints `GET`, `PUT` y `DELETE` para que lean, actualicen y anulen transacciones usando el `transaction_id`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Aandresf) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
