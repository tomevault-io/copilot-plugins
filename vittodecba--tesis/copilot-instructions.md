## tesis

> Sistema de gestión para producción de cerveza artesanal. Cubre recetas, planificación de lotes, fermentación y stock.

# AtonBeer - Tesis

Sistema de gestión para producción de cerveza artesanal. Cubre recetas, planificación de lotes, fermentación y stock.

## Stack Técnico

- **Backend:** .NET 8 Web API, Entity Framework Core 8, SQL Server
- **Frontend:** Angular 17+ (standalone components), TailwindCSS
- **Arquitectura:** Clean Architecture (Domain → Application → Infrastructure → API)

## Estructura del repositorio

```
Tesis/
├── AtonBeerTesis/          # Backend .NET
│   ├── AtonBeerTesis/              # API layer (Controllers, Program.cs)
│   ├── AtonBeerTesis.Application/  # Services, Interfaces, DTOs
│   ├── AtonBeerTesis.Domain/       # Entities, Enums, Interfaces
│   └── AtonBeerTesis.Infrastructure/ # DbContext, Repositories, Migrations
└── AtonBeerFront/          # Frontend Angular
    └── src/app/
        ├── components/     # Feature components
        ├── services/       # HTTP services
        └── Interfaces/     # TypeScript interfaces
```

## Comandos clave

```bash
# Backend
cd AtonBeerTesis
dotnet run --project AtonBeerTesis      # Puerto 5190
dotnet ef migrations add <Name> --project AtonBeerTesis.Infrastructure --startup-project AtonBeerTesis --configuration Release
dotnet ef database update --project AtonBeerTesis.Infrastructure --startup-project AtonBeerTesis --configuration Release

# Frontend
cd AtonBeerFront
npm install
ng serve    # Puerto 4200
```

> **Nota:** Usar `--configuration Release` en migraciones cuando VS tiene el proyecto en Debug abierto (bloquea DLLs).

## Módulos principales

### Recetas (`/recetas`)
- CRUD completo con ingredientes (`RecetaInsumo`) y pasos de elaboración
- Duplicado de recetas con versionado automático (V2, V3, V2.1…)
- Estilos de cerveza como strings en el campo `Estilo`
- **Al crear una receta con un estilo nuevo → se agrega automáticamente a todos los FormatosEnvase de stock**

### Lotes (`/planificacion`)
- Ciclo: Planificado → EnProceso → Finalizado / Descartado
- Vinculado a una Receta y un Fermentador
- **Sección de Designación de Volumen:** el usuario asigna litros a formatos de envase antes de finalizar
- **Al finalizar → el sistema genera ingresos de stock automáticamente** para cada designación
- Registros de fermentación diarios (pH, densidad, temperatura, presión)

### Stock (`/stock`)
- **FormatoEnvase:** tipos de envase con capacidad en litros (Barril 50L, Lata 473ml, etc.)
- **ProductoStock:** combinación FormatoEnvase × Estilo, con stock en unidades
- **MovimientoStock:** historial de ingresos/egresos con trazabilidad al lote

### Fermentadores
- Estados: Disponible, Ocupado, Sucio
- Un lote activo por fermentador a la vez

### Planificación de Producción
- Vincula Lote + Fermentador con fecha de inicio
- Se sincroniza con el estado del Lote al finalizar

### Clientes (`/clientes`)
- CRUD completo con filtros por tipo, ubicación y estado
- **TipoCliente:** `Franquicia` o `Externo`
- **EstadoCliente:** `Activo` o `Inactivo` (borrado lógico, no físico)
- CUIT validado y normalizado (`CuitValidator`) — no se puede cambiar con PATCH
- No se puede desactivar un cliente con pedidos activos (`EstadoId != 2 y != 4`)
- Campos de métricas: `UltimoPedido`, `UltimaCompra`, `TotalPedidos`
- Componente: `AtonBeerFront/src/app/components/clientes/clientes.component.ts`
- Servicio: `AtonBeerFront/src/app/services/clientes-api.ts`

### Pedidos (`/pedidos/registrar`)
- **Entidades:** `Pedido → DetallePedido → ProductoStock`, vinculado a `Cliente` y `EstadoPedido`
- **Estados (tabla catálogo):** `1=Pendiente`, `2=Entregado`, `4=Cancelado`
- **Flujo:** `Pendiente → Entregado` / `Pendiente → Cancelado` / `Entregado → Pendiente` (reversa)
- Solo pedidos `Pendiente` pueden editarse, cancelarse o entregarse
- **Al entregar:** descuenta stock (`MovimientoStock` tipo "Egreso") + asigna barriles físicos al cliente (`EstadoBarril.ConCliente`)
- **Reversa de entrega:** restaura stock + devuelve barriles a `EstadoBarril.Lleno`
- **Descuento Franquicia:** clientes tipo `Franquicia` reciben 10% de descuento (calculado en frontend)
- Solo clientes `Activo` aparecen en el selector al crear pedidos
- El UI tiene un estado "Facturado" en el flujo pero **aún no está implementado en el backend**
- Componente: `AtonBeerFront/src/app/components/registrar-pedido/registrar-pedido.ts`
- Servicio: `AtonBeerFront/src/app/core/services/pedido.service.ts`

#### Endpoints Pedidos
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/pedidos` | Lista todos |
| GET | `/api/pedidos/{id}` | Detalle para edición |
| POST | `/api/pedidos` | Crear (`PedidoCreacionDTO`) |
| PUT | `/api/pedidos` | Actualizar (`PedidoEdicionDTO`) |
| PATCH | `/api/pedidos/{id}/cancelar` | Cancelar |
| PATCH | `/api/pedidos/{id}/entregar` | Entregar (body: array barrilIds) |
| PUT | `/api/pedidos/{id}/deshacer-entrega` | Revertir entrega |

#### DTOs Pedidos
- **PedidoCreacionDTO:** `IdCliente, Observaciones, TotalPedido, Detalles[], FechaEntregaProgramada`
- **PedidoDetalleDTO:** `ProductoStockId, Cantidad, Precio, BarrilesAsignados[]`
- **PedidoEdicionDTO:** `Id, IdCliente, RazonSocial, Fecha, FechaEntregaProgramada, Observaciones, EstadoPedido, Detalles[], TotalPedido`
- **PedidoEntregadoDto:** `PedidoId, BarrilesIds[], Plazo, MetodoPago` *(Plazo y MetodoPago agregados para crear la Venta)*

### Ventas (`/ventas`)
- **Las ventas se crean automáticamente** al marcar un pedido como Entregado — no existe registro manual
- Una Venta por Pedido (índice único `PedidoId`)
- **NumeroVenta** generado automáticamente: `VNT-{año}-{Id:D5}` (ej: `VNT-2026-00001`)
- **EstadoVenta:** `Pendiente` (default al crear) o `Pagado`
- **MetodoPago:** `Efectivo` o `Transferencia` — el usuario lo elige al entregar el pedido o al editar
- **Plazo:** fecha de cobro — el usuario la elige al entregar el pedido o al editar
- **Edición:** campos modificables son `EstadoVenta`, `Plazo` y `MetodoPago`
- **Bloqueo:** una vez en estado `Pagado`, la venta no admite ninguna modificación (backend rechaza con 400)
- El botón "Editar" no aparece en filas con estado `Pagado` (UX)
- Componente: `AtonBeerFront/src/app/components/ventas/ventas.component.ts`
- Servicio: `AtonBeerFront/src/app/services/ventas.service.ts`

#### Endpoints Ventas
| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/api/ventas` | Lista todas las ventas |
| PATCH | `/api/ventas/{id}` | Editar (`PatchVentaDto`: EstadoVenta?, Plazo?, MetodoPago?) |

#### Campos Venta
`Id, NumeroVenta, FechaCreacion, ClienteId → Cliente, PedidoId → Pedido, MontoTotal, EstadoVenta (enum), Plazo, MetodoPago (enum)`

#### Flujo de entrega modificado
Al entregar un pedido, el usuario debe completar un modal "Datos de Venta" con Plazo y MetodoPago antes de confirmar. El endpoint `PATCH /api/pedidos/{id}/entregar` recibe `{ barrilesIds[], plazo, metodoPago }` (ya no es solo `number[]`).

## Reglas de negocio críticas

### Validaciones de Designación de Volumen
1. **División exacta:** `volumenAsignado % formato.CapacidadLitros == 0`  
   Ej: 40L en barriles de 50L → ERROR (no cabe exacto)
2. **No exceder producción:** `SUM(designaciones) <= lote.VolumenLitros`  
   Ej: 60L + 70L > 100L del lote → ERROR
3. **Estado del lote:** solo Planificado o EnProceso permiten modificar designaciones

### Generación automática de stock al finalizar lote
- Solo cuando `estadoFinal == Finalizado` (no Descartado)
- `unidades = designacion.VolumenAsignado / formato.CapacidadLitros`
- Busca `ProductoStock` por `FormatoEnvaseId` + `Estilo` del lote
- Registra `MovimientoStock` con `MotivoMovimiento = "Produccion"`

### Sincronización de estilos
- Al crear una Receta con un Estilo nuevo → `FormatoEnvaseService.AgregarEstiloATodosLosFormatosAsync(estilo)`
- Al crear un FormatoEnvase → se crean `ProductoStock` para todos los estilos existentes en Recetas

## Patrones de código

### Backend
- Repositories genéricos `IRepository<T>` para CRUD simple
- Repositories especializados (IRecetaRepository, ILoteRepository) para queries con Include
- DTOs en `Application/Dtos/` organizados por módulo (subcarpeta `STOCK/`, `LOTE/`, etc.)
- Servicios inyectados como `Scoped` en `Program.cs`

### Frontend
- Components standalone (Angular 17+)
- Services en `src/app/services/`
- Interfaces TypeScript en el mismo archivo del service (exportadas)
- TailwindCSS para estilos, sin CSS custom salvo animaciones

## Base de datos

**Tablas principales:**
- `Recetas`, `RecetaInsumos`, `PasosElaboracion`
- `Lotes`, `LoteDesignaciones`
- `Fermentadores`, `RegistrosFermentacion`
- `PlanificacionProduccion`
- `FormatosEnvase`, `ProductosStock`, `MovimientosStock`
- `Clientes`, `Pedidos`, `DetallesPedidos`, `EstadosPedido`
- `Ventas` (PedidoId único, FK Restrict a Cliente y Pedido)
- `ProductosPrueba` (legacy, no usar — reemplazado por ProductosStock)

**Última migración:** `StockRework` — agrega FormatosEnvase, ProductosStock, LoteDesignaciones; migra MovimientosStock a la nueva FK.

## Roles y permisos (RBAC frontend)

### Archivos clave
- **Constantes:** `AtonBeerFront/src/app/core/constants/roles.ts` — objeto `ROLES` con los nombres exactos de cada rol
- **Guard:** `AtonBeerFront/src/app/core/guards/role.guard.ts` — `roleGuard` funcional, lee `data.roles` de la ruta
- **AuthService:** método `hasRole(...roles: string[])` compara contra `rolNombre` del usuario en sesión
- **Layout:** `layout.ts` tiene helpers `puedeVerAdministracion()`, `puedeVerProduccion()`, etc. usados en `layout.html`

### Roles del sistema (IDs en BD)
`#1 Administrador` · `#3 Responsable de Planta` · `#4 Responsable de Pedidos` · `#5 Gerente` · `#6 Cocinero` · `#7 Gerente Mayor`

### Mapeo ruta → roles permitidos
| Ruta | Roles |
|---|---|
| `/inicio` | Todos |
| `/usuarios`, `/roles`, `/historial-accesos` | Administrador, Gerente |
| `/insumos`, `/stock`, `/recetas`, `/fermentadores` | Administrador, Resp. Planta, Cocinero |
| `/planificacion/**` | Administrador, Resp. Planta |
| `/barriles` | Administrador, Resp. Planta, Resp. Pedidos |
| `/clientes` | Administrador, Gerente, Resp. Pedidos |
| `/pedidos/registrar` | Administrador, Gerente, Resp. Planta, Resp. Pedidos |
| `/ventas` | Administrador, Gerente, Gerente Mayor |

---
> Source: [vittodecba/Tesis](https://github.com/vittodecba/Tesis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
