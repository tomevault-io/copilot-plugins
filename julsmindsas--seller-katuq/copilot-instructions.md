## seller-katuq

> Este documento proporciona instrucciones y contexto a GitHub Copilot para mejorar la asistencia de código inteligente en el proyecto Katuq Seller.

# Instrucciones para Cursor - Proyecto Katuq Seller

Este documento proporciona instrucciones y contexto a GitHub Copilot para mejorar la asistencia de código inteligente en el proyecto Katuq Seller.

## Estructura del Proyecto

Katuq Seller es una aplicación Angular 14 con una arquitectura modular. El proyecto sigue una estructura estándar de Angular con los siguientes componentes principales:

```
src/
  app/
    app-routing.module.ts    # Configuración de rutas principal
    app.module.ts            # Módulo principal
    app.component.ts         # Componente principal
    auth/                    # Autenticación y autorización
    components/              # Módulos y componentes funcionales
    pages/                   # Páginas de la aplicación
    pipes/                   # Pipes personalizados
    services/                # Servicios compartidos
    shared/                  # Componentes compartidos, interfaces, etc.
      shared.module.ts       # Módulo compartido
      components/            # Componentes reutilizables
      directives/            # Directivas personalizadas
      guard/                 # Guards de rutas
      models/                # Interfaces y modelos
      routes/                # Configuraciones de rutas adicionales
      services/              # Servicios compartidos
  assets/                    # Recursos estáticos
  environments/              # Configuración de entorno
```

## Módulos Principales

El proyecto se divide en múltiples módulos funcionales, cada uno representando una parte del sistema:

- **Dashboard**: Visualización de métricas y KPIs (`/dashboards`)
- **Seguridad**: Gestión de roles y usuarios (`/rol`, `/usuarios`)
- **Empresa**: Configuración de la empresa y módulos (`/empresas`)
- **Inventarios**: Gestión de productos y stock (`/inventario/catalogo`)
- **CRM**: Gestión de clientes (`/ventas/clientes`, `/ventas/clienteslista`)
- **Ventas**: Gestión de pedidos (`/ventas/crear-ventas`, `/ventas/pedidos`)
- **Producción**: Control de producción (`/produccion/dashboard`)
- **Despachos**: Gestión de entregas (`/despachos`)
- **POS**: Punto de venta (`/pos/ventas`)
- **Soporte**: Sistema de tickets y asistencia

## Tecnologías y Dependencias

### Framework y Versiones
- Angular 14.1.x
- TypeScript 4.7.x
- Node.js (recomendado 16.x)

### Principales Dependencias
- **PrimeNG 14.2.x**: Framework UI para componentes
- **ng-bootstrap 13.x**: Integración con Bootstrap
- **Firebase 9.17.x**: Servicios de backend y storage
- **ngx-datatable 20.x**: Para tablas de datos complejas
- **Chart.js/ApexCharts**: Visualización de datos

### Estilos
- SCSS como preprocesador
- Bootstrap 5.2.x como framework base
- PrimeIcons para iconografía

## Patrones de Código y Convenciones

### Estructura de Módulos
Cada módulo funcional sigue una estructura similar:

```
module-name/
  module-name.component.ts       # Componente principal
  module-name.component.html     # Plantilla
  module-name.component.scss     # Estilos específicos
  module-name.module.ts          # Definición del módulo
  module-name-routing.module.ts  # Configuración de rutas
  subcomponents/                 # Subcomponentes (si aplica)
```

### Convenciones de Nombramiento
- Componentes: PascalCase + sufijo Component (ej: `ProductsComponent`)
- Servicios: PascalCase + sufijo Service (ej: `ProductsService`)
- Interfaces: PascalCase (ej: `Product`)
- Archivos: kebab-case (ej: `products.service.ts`)

### Patrones de Servicios
- Servicios para lógica de negocio y acceso a datos
- Inyección de dependencias a través de constructores
- Patrones reactivos con RxJS

## Instrucciones Específicas para Copilot

### Implementación de Nuevos Componentes
Al crear nuevos componentes:

1. Usar la CLI de Angular: `ng generate component path/to/component`
2. Seguir la estructura modular existente
3. Actualizar el módulo correspondiente
4. Añadir rutas si es necesario

### Estilo y UI
- Aprovechar PrimeNG para componentes UI complejos
- Mantener coherencia con estilos existentes
- Utilizar variables SCSS del tema

### Manejo de Datos y Estado
- Usar RxJS para flujos de datos reactivos
- Servicios para compartir estado entre componentes
- Evitar mutaciones directas de estado

### Integración con Backend
- Usar HttpClient para peticiones API
- Manejar errores y loading states
- Considerar Firebase para almacenamiento cuando aplique

## Funcionalidades Específicas

### Sistema de Autenticación
- Guards para protección de rutas (`AuthGuard`, `AdminGuard`)
- Servicios de autenticación y autorización

### Internacionalización
- Soporte para múltiples idiomas con ngx-translate
- Archivos de traducción en `/assets/i18n/`

### Módulos Lazy-Loaded
- Todos los módulos principales utilizan carga perezosa
- Configurados en app-routing.module.ts y routes.ts

## Buenas Prácticas

1. **Performance**: 
   - Minimizar suscripciones a observables
   - Usar `trackBy` para listas
   - Evitar detección de cambios innecesaria

2. **Seguridad**:
   - Validar inputs del usuario
   - Sanitizar datos HTML dinámicos
   - Implementar protección CSRF

3. **Mantenibilidad**:
   - Comentar código complejo
   - Mantener componentes pequeños y enfocados
   - Escribir tests para funcionalidad crítica

## Problemas Conocidos y Limitaciones

- Algunos componentes usan `@ts-nocheck` por complejidad en la migración de tipos
- Firebase requiere configuración específica en environment.ts

---

Esta documentación se actualizará según evolucione el proyecto.

# Contexto y Scaffolding de Modelos Principales

## Scaffolding de src/app/shared/models/productos/Producto.ts

```typescript
// ...existing code...
export interface Producto {
  dimensiones?: Dimensiones;
  disponibilidad?: Disponibilidad;
  marketplace?: Marketplace;
  exposicion?: Exposicion;
  categorias?: Categoria;
  identificacion?: Identificacion;
  procesoComercial?: ProcesoComercial;
  ciudades?: Ciudades;
  cd?: string;
  crearProducto?: CrearProducto;
  precio?: Precio;
  date_edit?: string;
  variableForm?: string;
  rating?: number;
  otrosProcesos?: OtrosProcesos;
  bodegaId?: string; // Relaciona el producto con una bodega
}

export interface ProductoCarrito {
  dimensiones: Dimensiones;
  disponibilidad: Disponibilidad;
  exposicion: Exposicion;
  categorias: Categoria;
  identificacion: Identificacion;
  cd: string;
  crearProducto: CrearProducto;
  precio: Precio;
  date_edit: string;
  variableForm: string;
  rating: number;
  bodegaId?: string;
}
// ...existing code...
```

## Scaffolding de src/app/components/ventas/modelo/pedido.ts

```typescript
// ...existing code...
export interface Pedido {
    transaccionId?: any;
    entregado?: UserLite;
    transportador?: any;
    nroShippingOrder?: string;
    despachador?: UserLite;
    fechaYHorarioDespachado?: string;
    _id?: string;
    fechaHoraEmpacado?: string;
    porceDescuento?: number;
    nroPedido?: string;
    empacador?: string;
    referencia: string;
    nroPedidoReferencia?: string;
    company?: string;
    cliente?: Cliente;
    notasPedido?: NotasPedido;
    carrito?: Carrito[];
    formaDePago?: string;
    cuponAplicado?: string;
    totalPedidoSinDescuento?: number;
    totalEnvio?: number;
    totalDescuento?: number;
    totalImpuesto?: number;
    anticipo?: number;
    faltaPorPagar?: number;
    subtotal?: number;
    totalPedididoConDescuento?: number;
    facturacion?: Facturacion;
    envio?: Envio;
    fechaEntrega?: string;
    horarioEntrega?: string;
    formaEntrega?: string;
    asesorAsignado?: UserLite;
    fechaCreacion?: string;
    estadoProceso: EstadoProceso;
    shippingOrder?: any;
    estadoPago: EstadoPago;
    nroFactura?: string;
    fechaFactura?: string;
    PagosAsentados?: Pago[];
    validacion?: boolean;
    pagoInformation?: PagoInformation;
}
// ...existing code...
```

> Estos modelos son fundamentales para la gestión de productos y pedidos en el sistema. Se recomienda mantenerlos actualizados y documentados, siguiendo las convenciones de nomenclatura y estructura del proyecto.

---

# Scaffolding de src

```
src/
├── app/                              # Directorio principal de la aplicación
│   ├── app-routing.module.ts         # Configuración de rutas principal
│   ├── app.component.ts/html/scss    # Componente principal
│   ├── app.module.ts                 # Módulo principal con imports
│   ├── auth/                         # Autenticación
│   │   └── login/                    # Componente de login
│   ├── components/                   # Componentes funcionales
│   │   ├── change-password/          # Cambio de contraseña
│   │   ├── chat/                     # Sistema de chat
│   │   ├── dashboard/                # Dashboard principal
│   │   ├── despachos/                # Gestión de despachos
│   │   ├── diagnostic-survey/        # Encuesta diagnóstico
│   │   ├── ecomerce/                 # Funcionalidad ecommerce
│   │   ├── empresas/                 # Gestión de empresas
│   │   │   ├── crearEmpresa/         # Crear empresa
│   │   │   ├── modulovariable/       # Módulos variables
│   │   │   │   └── produccion/       # Producción
│   │   │   │       ├── procesos/     # Procesos
│   │   │   │       └── centrotrabajos/  # Centros de trabajo
│   │   ├── inventarios/              # Gestión de inventario
│   │   ├── loader/                   # Componente de carga
│   │   ├── maestros/                 # Datos maestros
│   │   ├── payment-callback/         # Callback pagos
│   │   ├── pos/                      # Punto de venta
│   │   ├── proceso/                  # Procesos
│   │   │   ├── canales/              # Canales
│   │   │   ├── generos/              # Géneros
│   │   │   ├── ocasiones/            # Ocasiones
│   │   │   └── variables/            # Variables
│   │   ├── produccion/               # Producción
│   │   │   └── dashboard/            # Dashboard producción
│   │   │       ├── tracking/         # Seguimiento
│   │   │       └── cerrararticulo/   # Cierre artículos
│   │   ├── productos/                # Gestión productos
│   │   │   └── crear-productos/      # Crear productos
│   │   ├── prospect-manager/         # Gestión de prospectos
│   │   ├── rol/                      # Gestión de roles
│   │   │   ├── roles/                # Componente roles
│   │   │   └── crear-rol/            # Crear rol
│   │   ├── soporte/                  # Sistema de soporte
│   │   │   ├── mis-tickets/          # Tickets soporte
│   │   │   └── mis-ideas/            # Ideas
│   │   ├── terms-conditions/         # Términos y condiciones
│   │   ├── usuarios/                 # Gestión usuarios
│   │   └── ventas/                   # Sistema de ventas
│   ├── pages/                        # Páginas principales
│   │   └── authentication/           # Autenticación
│   ├── pipes/                        # Pipes personalizados
│   │   └── safe-url.pipe.ts          # Pipe URL seguras
│   ├── services/                     # Servicios principales
│   │   ├── analytics.service.ts      # Servicio analítica
│   │   ├── companies.service.ts      # Servicio empresas
│   │   └── user.service.ts           # Servicio usuarios
│   ├── shared/                       # Componentes compartidos
│   │   ├── components/               # Componentes comunes
│   │   │   ├── breadcrumb/           # Migas de pan
│   │   │   ├── floating-button/      # Botón flotante
│   │   │   ├── header/               # Cabecera
│   │   │   ├── loader/               # Loader
│   │   │   ├── sidebar/              # Barra lateral
│   │   │   └── chat-form/            # Formulario chat
│   │   ├── directives/               # Directivas personalizadas
│   │   ├── guard/                    # Guards de rutas
│   │   │   └── admin.guard.ts        # Guard admin
│   │   ├── models/                   # Modelos de datos
│   │   │   ├── productos/            # Modelos productos
│   │   │   │   └── Producto.ts       # Modelo producto
│   │   │   └── User/                 # Modelos usuarios
│   │   ├── routes/                   # Configuración rutas
│   │   │   └── routes.ts             # Rutas contenido
│   │   └── services/                 # Servicios compartidos
│   │       ├── errores/              # Manejo errores
│   │       ├── firebase/             # Firebase
│   │       ├── interceptor/          # Interceptores HTTP
│   │       ├── katuqintelligence/    # IA Katuq
│   │       ├── maestros/             # Maestros
│   │       │   └── maestro.service.ts # Servicio maestros
│   │       └── ventas/               # Servicios ventas
│   └── welcome/                      # Pantalla bienvenida
└── environments/                     # Configuración entornos
    ├── environment.ts                # Desarrollo
    └── environment.prod.ts           # Producción
```

> Esta estructura define la organización principal del código fuente y facilita la escalabilidad y mantenibilidad del proyecto.



---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julsmindsas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/julsmindsas)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
