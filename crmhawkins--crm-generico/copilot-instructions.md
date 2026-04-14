## crm-generico

> - nombre: string (required)

# ==========================
# MÓDULO: CATEGORÍAS DE PRODUCTOS
# ==========================

CategoriaProducto:
  fields:
    - nombre: string (required)
    - descripcion: text (nullable)
    - oferta_porcentaje: decimal(5,2) (nullable, representa % de descuento aplicado a todos los productos)
  relationships:
    - hasMany: Producto
  features:
    - CRUD completo en el panel de administración
    - Campo de oferta opcional que afecta a todos los productos de esa categoría
    - Listado filtrable por nombre o con/sin oferta activa

# ==========================
# MÓDULO: PRODUCTOS
# ==========================

Producto:
  fields:
    - categoria_id: foreignId -> CategoriaProducto
    - nombre: string (required)
    - imagen: string (nullable)
    - codigo: string (unique)
    - precio: decimal(10,2)
    - descuento_porcentaje: decimal(5,2) (nullable)
    - fecha_inicio_descuento: date (nullable)
    - fecha_fin_descuento: date (nullable)
  relationships:
    - belongsTo: CategoriaProducto
  features:
    - CRUD con upload de imagen
    - Cálculo automático del precio final si hay descuento activo (fecha_inicio <= hoy <= fecha_fin)
    - Posibilidad de aplicar descuentos globales por categoría
    - Buscador en tiempo real (por código o nombre)
    - Listado con imagen y estado de oferta

# ==========================
# MÓDULO: TPV
# ==========================

TPV:
  features:
    - Abrir caja:
        descripción: "Solicita el importe inicial del cajón al inicio del turno."
        modelo: Caja
        campos:
          - fecha_apertura: datetime
          - fondo_inicial: decimal(10,2)
          - usuario_id: foreignId -> User
          - estado: enum('abierta','cerrada')
    - Cerrar caja:
        descripción: "Calcula automáticamente ventas, total cobrado, beneficios y genera resumen."
        salida:
          - total_ventas
          - total_beneficio
          - total_tickets
          - fondo_final (fondo_inicial + ventas)
    - Procesar venta:
        modelo: Venta / Ticket
        campos:
          - caja_id: foreignId -> Caja
          - productos: json (id_producto, cantidad, precio_unitario, subtotal)
          - total: decimal(10,2)
          - metodo_pago: enum('efectivo','tarjeta')
          - estado: enum('completada','standby')
          - fecha: datetime
        relaciones:
          - belongsTo: Caja
          - belongsToMany: Producto (pivot con cantidad y precio)
        funcionalidades:
          - Añadir productos desde interfaz visual (por categoría o buscador)
          - Botón "Standby" → guarda ticket en estado standby
          - Botón "Reanudar" → carga ticket standby para continuar venta
          - Generar ticket PDF o impreso (nombre empresa, fecha, productos, totales)
    - Interfaz:
        descripción: "Diseño tipo POS táctil, con botones grandes de categorías y productos."
        stack:
          - Blade + Alpine.js (o Vue si ya está en uso)
        comportamiento:
          - Mostrar categorías en la parte superior o lateral
          - Mostrar productos filtrados por categoría
          - Panel lateral con los productos añadidos al carrito
          - Botones: Cancelar venta, Guardar en standby, Finalizar venta
          - Modal de apertura/cierre de caja

# ==========================
# FLUJO DE DATOS Y LÓGICA
# ==========================

workflow:
  1. Usuario abre caja → se guarda registro `caja` (estado abierta).
  2. Usuario realiza ventas → se crean registros `tickets` vinculados a `caja`.
  3. Puede guardar una venta en “standby” → ticket en estado `standby`.
  4. Puede reanudar venta en standby → cambia a `completada` al finalizar.
  5. Al cerrar caja → se calculan totales, beneficios y se guarda `estado=cerrada`.

# ==========================
# MIGRACIONES Y MODELOS NUEVOS
# ==========================

migrations_to_add:
  - create_categorias_productos_table
  - create_productos_table
  - create_cajas_table
  - create_tickets_table
  - create_ticket_producto_pivot_table

# ==========================
# TESTS Y VERIFICACIÓN
# ==========================

testing:
  - Añadir tests unitarios para cada modelo
  - Tests de integración: apertura/cierre de caja, standby, descuentos activos
  - Tests de UI: flujo completo de compra en TPV

# ==========================
# ESTILO Y NOMENCLATURA
# ==========================

naming:
  controllers:
    - CategoriaProductoController
    - ProductoController
    - TpvController
    - CajaController
    - TicketController
  views:
    - resources/views/tienda/categorias/*.blade.php
    - resources/views/tienda/productos/*.blade.php
    - resources/views/tpv/*.blade.php
  routes:
    prefix: /tienda y /tpv
    middleware: auth

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/crmhawkins) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
