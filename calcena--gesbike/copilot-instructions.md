## gesbike

> GesBike es una aplicación web para la gestión integral de mantenimiento de bicicletas. Permite gestionar vehículos (bicicletas), mantenimientos, recambios, compras, rutas, grupos y operaciones de mantenimiento. El sistema está diseñado para que usuarios particulares o pequeños talleres puedan llevar un control exhaustivo de sus bicicletas y sus mantenimientos.

# GesBike - Sistema de Gestión de Mantenimiento de Bicicletas

## 1. Descripción General

GesBike es una aplicación web para la gestión integral de mantenimiento de bicicletas. Permite gestionar vehículos (bicicletas), mantenimientos, recambios, compras, rutas, grupos y operaciones de mantenimiento. El sistema está diseñado para que usuarios particulares o pequeños talleres puedan llevar un control exhaustivo de sus bicicletas y sus mantenimientos.

## 2. Arquitectura

El proyecto sigue el patrón **MVC (Model-View-Controller)** con una capa adicional de **Repositories** para el acceso a datos. También cuenta con una capa de **API** para comunicación asíncrona y una capa de **Servicios JavaScript** para el frontend.

### Capas de la Arquitectura

```
┌─────────────────────────────────────────────┐
│         Views (Vistas PHP + HTML)           │
│     Listado, formulario, dashboard           │
├─────────────────────────────────────────────┤
│        Services (JS Frontend)               │
│   Axios calls a API, render dinámico        │
├─────────────────────────────────────────────┤
│              API (Endpoints)                │
│    api/{recurso}/{recurso}.php?{action}     │
├─────────────────────────────────────────────┤
│            Controllers (PHP)                │
│    Lógica de control, validación, flujo     │
├─────────────────────────────────────────────┤
│             Models (PHP)                    │
│    Wrappers que llaman a Repositories       │
├─────────────────────────────────────────────┤
│           Repositories (PHP)                │
│    Queries SQL, acceso a datos (PDO)        │
├─────────────────────────────────────────────┤
│        DatabaseConnection (SQLite PDO)      │
└─────────────────────────────────────────────┘
```

### Flujo de una operación típica (ej: crear vehículo)

```
Usuario → View (form.php) → JS (vehiculo.js: guardarVehiculo())
  → POST a API (api/vehiculos/vehiculo.php?nuevoVehiculo)
    → Controller (controllers/vehiculo.php: handle_nuevo_vehiculo())
      → Model (models/vehiculo.php: nuevoVehiculo())
        → Repository (repositories/vehiculo.php: nuevo_vehiculo_repo())
          → SQL INSERT → SQLite DB
```

## 3. Estructura de Directorios

```
gesBike/
├── api/                          # Endpoints API REST
│   ├── compras/compra.php
│   ├── grupos/grupo.php
│   ├── helpers/helper.php        # Endpoints genéricos (uploadFile, getGrupos, getKilometros...)
│   ├── login/login.php
│   ├── log/log.php
│   ├── mantenimientos/mantenimiento.php
│   ├── recambios/recambio.php
│   ├── rutas/ruta.php
│   ├── stocks/stock.php
│   └── vehiculos/vehiculo.php
├── assets/                       # Recursos estáticos
│   ├── css/
│   │   ├── bootstrap/
│   │   ├── compras/
│   │   ├── detalles/
│   │   ├── grupos/
│   │   ├── login/
│   │   ├── main/
│   │   ├── mantenimientos/
│   │   ├── recambios/
│   │   ├── rutas/
│   │   ├── stocks/
│   │   ├── vehiculos/
│   │   ├── theme.css             # 70+ variables CSS para modo claro/oscuro
│   │   └── style.css             # Estilos globales compartidos
│   ├── js/
│   │   ├── axios/
│   │   └── bootstrap/
│   └── images/
│       ├── icons/
│       │   ├── Grupos/           # 22 iconos predefinidos para grupos
│       │   └── Localizaciones/   # Iconos de localizaciones (delante, detras, etc.)
│       ├── Recambios/            # Imágenes subidas de recambios (UUID)
│       └── Vehiculos/            # Imágenes subidas de bicicletas (UUID)
├── attachments/                  # Archivos adjuntos a mantenimientos
├── controllers/                  # Controladores PHP
│   ├── attach.php                # Subida de archivos e imágenes + compressImage()
│   ├── compra.php
│   ├── grupo.php                 # CRUD completo (getList, getById, nuevo, editar, eliminar)
│   ├── helper.php
│   ├── login.php                 # Autenticación y persistencia de tema
│   ├── log.php
│   ├── mantenimiento.php         # CRUD + getKmsByGrupo + getHistorico
│   ├── recambio.php
│   ├── ruta.php
│   ├── selector.php
│   ├── stock.php
│   ├── translate.php
│   └── vehiculo.php              # CRUD + subida imagen
├── database/
│   ├── app.db                    # Archivo SQLite principal
│   ├── gesbike.db
│   ├── DatabaseConnection.php    # Conexión PDO
│   └── backups/                  # Copias de seguridad
├── helpers/
│   ├── backup.php
│   ├── config.php                # Carga de variables de entorno (.env)
│   └── helper.php                # Funciones auxiliares PHP (random_file_enumerator, new_guui_generator, etc.)
├── jobs/
│   └── cron_email.php            # Tareas programadas (SMTP Gmail)
├── models/                       # Modelos PHP (wrappers)
│   ├── attach.php
│   ├── compra.php
│   ├── grupo.php
│   ├── helper.php
│   ├── login.php
│   ├── log.php
│   ├── mantenimiento.php
│   ├── recambio.php
│   ├── ruta.php
│   ├── selector.php
│   ├── stock.php
│   ├── translate.php
│   └── vehiculo.php
├── repositories/                 # Repositorios (queries SQL)
│   ├── attach.php
│   ├── compra.php
│   ├── grupo.php
│   ├── helper.php
│   ├── login.php
│   ├── log.php
│   ├── mantenimiento.php
│   ├── recambio.php
│   ├── ruta.php
│   ├── selector.php
│   ├── stock.php
│   ├── translate.php
│   └── vehiculo.php              # LEFT JOIN con ultimos_kms
├── services/                     # Servicios JavaScript frontend
│   ├── compras/compra.js
│   ├── componentes/sitebar.js    # Menú lateral con navegación
│   ├── detalles/detalle.js       # Vista detalles: pestaña km + histórico
│   ├── grupos/grupo.js           # CRUD grupos
│   ├── helpers/helper.js         # Utilidades compartidas (formatFechaISO, getGrupos, selectContainsText...)
│   ├── login/login.js            # Autenticación y autologin
│   ├── logs/logs.js
│   ├── main/main.js              # Dashboard principal
│   ├── mantenimientos/mantenimiento.js
│   ├── recambios/recambio.js
│   ├── rutas/ruta.js
│   ├── stocks/stock.js
│   ├── theme/theme.js            # Cambio de modo claro/oscuro con persistencia
│   └── translate/translate.js
├── views/                        # Vistas PHP
│   ├── components/
│   │   ├── footer.php
│   │   ├── header_info.php
│   │   ├── menu.php
│   │   ├── menu_actions.php
│   │   └── sidebar.php           # Menú lateral con theme toggle y navegación
│   ├── compras/
│   │   ├── main.php              # Listado de compras
│   │   └── compra.php            # Alta/edición
│   ├── detalles/detalle.php      # Detalle vehículo: 2 pestañas (km + histórico)
│   ├── grupos/
│   │   ├── main.php              # Listado con cards + FAB
│   │   └── form.php              # Alta/edición con selector de icono grid
│   ├── mantenimientos/mantenimiento.php  # 3 pestañas (formulario, observaciones, adjuntos)
│   ├── recambios/
│   │   ├── main.php              # Listado con FAB + selector vehículo
│   │   └── recambio.php          # Alta/edición con subida de imagen
│   ├── rutas/ruta.php
│   ├── stocks/stock.php
│   ├── vehiculos/
│   │   ├── vehiculo.php          # Listado con totalizador km + FAB
│   │   └── form.php              # Alta/edición con subida de imagen
│   └── main.php                  # Dashboard principal
├── tests/
├── photos/
├── index.php                     # Login moderno con card, gradientes, iconos, autologin
├── .env                          # Variables de entorno
└── compress_existing_images.php
```

## 4. Tecnologías Utilizadas

### Backend
- **PHP 7.4+** - Lenguaje del servidor
- **SQLite** - Base de datos embebida
- **PDO** - Acceso a datos

### Frontend
- **HTML5** - Estructura
- **CSS3** - Estilos (Bootstrap 5 + CSS variables para theming)
- **JavaScript (ES6+)** - Interactividad
- **Axios** - Cliente HTTP
- **SweetAlert2** - Alertas y diálogos
- **FontAwesome 6** - Iconografía
- **Hammer.js** - Gestos táctiles (swipe)

### Herramientas
- **Git** - Control de versiones
- **PHP Sessions** - Gestión de autenticación

## 5. Configuración

### Variables de Entorno (.env)

```env
APP_ENV=local
APP_VERSION=1.0-0
APP_NAME=Mantenimientos bicicletas
DB_TYPE=sqlite
DB_PATH=../../database/app.db
```

### Cache-Busting de Assets

Todos los CSS/JS incluyen `?<?php random_file_enumerator() ?>` que genera un timestamp único (`time()`) para que el navegador no sirva archivos cacheados tras cada cambio. Definido en `helpers/helper.php`.

### Puntos de Entrada

- **`index.php`** - Pantalla de login (gradiente púrpura #667eea→#764ba2, card centrada, animación fadeInUp)
- **`views/main.php`** - Dashboard principal (requiere autenticación)
- **`api/*/**`** - Endpoints de la API

## 6. Base de Datos

### Esquema Completo de Tablas

#### usuarios
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| rol_id | INTEGER | FK a roles |
| nombre | TEXT | Nombre de usuario |
| password | TEXT | Contraseña |
| activo | INTEGER | Estado lógico |
| theme | TEXT | Preferencia: 'light' o 'dark' |

#### vehiculos
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| fecha_compra | TEXT | Fecha de adquisición |
| anagrama | TEXT | Identificador corto |
| nombre | TEXT | Nombre de la bicicleta |
| kms_inicio | INTEGER | KMs iniciales |
| imagen | TEXT | Nombre archivo en assets/images/Vehiculos/ |
| observaciones | TEXT | Notas adicionales |
| puntero | TEXT | Icono (bullet_nombre.png) |
| categoria | TEXT | 'pulmonar' o 'electrica' |
| usuario_id | INTEGER | FK a usuarios |
| is_active | INTEGER | 1=activo, 0=inactivo |
| created_at | TEXT | Fecha creación |
| modified_at | TEXT | Fecha modificación |
| deleted_at | TEXT | Fecha borrado lógico |

#### ultimos_kms
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| vehiculo_id | INTEGER | FK única a vehiculos |
| kms | INTEGER | Últimos kilómetros registrados |
| fecha_actualizacion | DATETIME | Timestamp última actualización |

#### grupos
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| nombre | TEXT | Nombre del grupo |
| imagen | TEXT | Icono (assets/images/icons/Grupos/*.png) |
| agrupador_id | INTEGER | FK a grupo padre (0=ninguno) |
| trazabilidad | INTEGER | 1=con trazabilidad, 0=sin |
| vista_resumen | INTEGER | 1=visible en resumen |
| is_active | INTEGER | 1=activo, 0=inactivo |
| created_at | TEXT | |
| modified_at | TEXT | |
| deleted_at | TEXT | |

#### recambios
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| nombre | TEXT | Nombre del recambio |
| referencia | TEXT | Número de referencia |
| grupo_id | INTEGER | FK a grupos |
| vehiculo_id | INTEGER | FK a vehiculos |
| imagen | TEXT | Ruta en assets/images/Recambios/ |
| observaciones | TEXT | Notas |
| is_active | INTEGER | 1=activo, 0=inactivo |
| created_at | TEXT | |
| modified_at | TEXT | |
| deleted_at | TEXT | |

#### mantenimientos
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| vehiculo_id | INTEGER | FK a vehiculos |
| fecha | TEXT | Fecha del mantenimiento |
| kms | INTEGER | KMs en el momento |
| operacion_id | INTEGER | FK a operaciones |
| recambio_id | INTEGER | FK a recambios |
| grupo_id | INTEGER | FK a grupos |
| localizacion_id | INTEGER | FK a localizaciones |
| Precio | TEXT | Costo |
| Unidades | INTEGER | Cantidad |
| observaciones | TEXT | Notas |
| motor_id | INTEGER | FK a motores |
| is_active | INTEGER | 1=activo |
| created_at | TEXT | |
| modified_at | TEXT | |
| deleted_at | TEXT | |

#### compras
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| recambio_id | INTEGER | FK a recambios |
| proveedor | TEXT | Nombre del proveedor |
| precio | REAL | Costo unitario |
| unidades | INTEGER | Cantidad comprada |
| fecha | TEXT | Fecha de compra |
| is_active | INTEGER | 1=activo |

#### rutas
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| vehiculo_id | INTEGER | FK a vehiculos |
| fecha_inicio | DATETIME | Inicio de ruta |
| fecha_fin | DATETIME | Fin de ruta |
| tiempo_total | TEXT | Duración total |
| kms | DECIMAL | Kilómetros |
| metros_ascenso | INTEGER | Desnivel positivo |
| velocidad_media | DECIMAL | Velocidad promedio |
| potencia_promedio_w | INTEGER | Potencia media |

#### operaciones
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| nombre | TEXT | Nombre (Engrasado, Limpieza, Sustitución, Medición, Taller, Mecánica general) |
| imagen | TEXT | Icono PNG |
| is_active | INTEGER | |

#### localizaciones
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| nombre | TEXT | delante, detras, derecha, izquierda |
| imagen | TEXT | Icono PNG |
| grupo_id | INTEGER | FK a grupos (0=genérico) |

#### motores
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| vehiculo_id | INTEGER | FK a vehiculos |
| kms | INTEGER | Horas/km del motor |
| is_active | INTEGER | |

#### adjuntos
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| guid | TEXT | UUID del archivo |
| nombre_original | TEXT | Nombre original |
| ruta | TEXT | Ruta en attachments/ |
| vehiculo_id | INTEGER | FK a vehiculos |
| mantenimiento_id | INTEGER | FK a mantenimientos |
| created_at | TEXT | |

#### logs
| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | INTEGER PK | Autoincremental |
| usuario_id | INTEGER | FK a usuarios |
| accion | TEXT | Acción realizada |
| created_at | TEXT | |

## 7. API - Endpoints

Los endpoints siguen el patrón `api/{recurso}/{recurso}.php?{action}`. Todas las acciones vía POST con JSON body.

### Vehículos (`api/vehiculos/vehiculo.php`)

| Acción | Descripción |
|--------|-------------|
| getVehiculos | Lista vehículos del usuario (LEFT JOIN ultimos_kms) |
| getVehiculoById | Obtiene vehículo por ID |
| nuevoVehiculo | Crea nuevo vehículo |
| editarVehiculo | Actualiza vehículo |
| eliminarVehiculo | Borrado lógico (is_active=0) |
| getMotorVehiculo | Datos del motor |
| uploadVehiculoImage | Sube imagen a assets/images/Vehiculos/ |

### Grupos (`api/grupos/grupo.php`)

| Acción | Descripción |
|--------|-------------|
| getListGrupos | Lista todos los grupos activos |
| getGrupoById | Obtiene grupo por ID |
| nuevoGrupo | Crea nuevo grupo |
| editarGrupo | Actualiza grupo |
| eliminarGrupo | Borrado lógico (is_active=0) |

### Recambios (`api/recambios/recambio.php`)

| Acción | Descripción |
|--------|-------------|
| getListRecambios | Lista recambios por vehiculo_id (con stock calculado) |
| getRecambioById | Obtiene recambio por ID |
| nuevoRecambio | Crea nuevo recambio |
| editarRecambio | Actualiza recambio |
| eliminarRecambio | Borrado lógico |
| uploadRecambioImage | Sube imagen a assets/images/Recambios/ |

### Compras (`api/compras/compra.php`)

| Acción | Descripción |
|--------|-------------|
| getListCompras | Lista compras por recambio_id |
| nuevaCompra | Crea nueva compra |
| editarCompra | Actualiza compra |
| eliminarCompra | Borrado lógico |

### Mantenimientos (`api/mantenimientos/mantenimiento.php`)

| Acción | Descripción |
|--------|-------------|
| getListMantenimientos | Lista mantenimientos por vehiculo_id |
| createNewMantenimiento | Crea nuevo mantenimiento |
| getListAttachments | Lista adjuntos por mantenimiento_id |
| deleteAttachment | Elimina adjunto físico y de BD |
| getMantenimientosById | Obtiene mantenimiento por ID |
| deleteMantenimiento | Elimina mantenimiento y sus adjuntos |
| editarMantenimiento | Actualiza mantenimiento |
| getKmsByGrupo | Resumen km agrupado por localización |
| getHistorico | Histórico detallado por grupo (con duración pieza, operación, recambio, etc.) |

### Helpers (`api/helpers/helper.php`)

| Acción | Descripción |
|--------|-------------|
| getGrupos | Lista grupos (para selectores) |
| getKilometrosByVehiculo | Obtiene km actuales del vehículo |
| setKilometrosByVehiculo | Actualiza km actuales en ultimos_kms |
| uploadFile | Sube archivo adjunto (source=adjunto) |

### Login (`api/login/login.php`)

| Acción | Descripción |
|--------|-------------|
| auth | Autenticación, devuelve id + theme del usuario |
| setTheme | Guarda preferencia de tema (light/dark) |

### Stocks / Rutas / Log

| Acción | Endpoint | Descripción |
|--------|----------|-------------|
| getStocks | api/stocks/stock.php?getStocks | Lista stock bajo mínimo |
| getRutas | api/rutas/ruta.php?getRutas | Lista rutas |
| getLogs | api/log/log.php?getLogs | Histórico de acciones |

## 8. Módulos y Flujos de Trabajo

### 8.1 Autenticación

1. Usuario accede a `index.php`
2. Login moderno: card centrada con gradiente púrpura, iconos en inputs, animación fadeInUp
3. JavaScript llama a `api/login/login.php?auth`
4. Sistema valida contra tabla `usuarios`, crea sesión PHP
5. Recupera `theme` del usuario y lo guarda en sessionStorage
6. Muestra "¡Bienvenido {nombre}!" en púrpura (`--login-message`) y redirige a `views/main.php`
7. **Autologin**: detecta credenciales precargadas y ejecuta auth automáticamente (vía setInterval, touchstart, visibilitychange)

### 8.2 Sistema de Temas (Claro / Oscuro)

1. Cada usuario tiene su preferencia almacenada en `usuarios.theme`
2. Al loguearse, el tema se carga desde la BD y se guarda en sessionStorage
3. `initTheme()` en cada página aplica `data-theme="dark|light"` en `<html>`
4. Botón de cambio en el menú lateral (icono luna/sol)
5. `services/theme/theme.js` maneja toggle, persistencia vía API
6. `--login-bg`: gradiente púrpura intenso. `--body-bg`: gradiente púrpura muy suave (`#f0ecf9`→`#f8f4ff`) en claro, `#1a1a2e` en oscuro
7. Todas las vistas incluyen `theme.css` y `theme.js`
8. Transiciones suaves (0.3s ease) en colores de fondo, texto, bordes

### 8.3 Gestión de Vehículos

**Listado** (`views/vehiculos/vehiculo.php`):
- Cards con imagen (100x70px, object-fit: cover), nombre, anagrama, badge activo (verde) / inactivo (rojo), fecha compra, km actuales
- **Totalizador km**: tarjeta superior con suma de km de todos los vehículos
- Tap en card abre offcanvas (editar / eliminar)
- FAB flotante con gradiente corporativo

**Formulario** (`views/vehiculos/form.php`):
- Subida de imagen: contenedor dashed con icono cámara + "Tocar para añadir foto" (`upload-container`, `upload-preview`, `upload-placeholder`)
- Input `accept="image/*"` (cámara o galería en móvil)
- Campos: nombre, anagrama, fecha compra, km iniciales, categoría (pulmonar/electrica), observaciones
- Imagen se sube vía `api/vehiculos/vehiculo.php?uploadVehiculoImage` a `assets/images/Vehiculos/` con UUID

### 8.4 Gestión de Grupos

**Listado** (`views/grupos/main.php`):
- Cards con icono del grupo (48x48px desde `assets/images/icons/Grupos/`) + nombre + badge trazabilidad
- Tap en card abre offcanvas (editar / eliminar)
- FAB flotante para nuevo grupo

**Formulario** (`views/grupos/form.php`):
- Nombre, selector de icono (grid de 22 iconos predefinidos con borde seleccionable), agrupador padre (dropdown), switches trazabilidad y vista resumen

### 8.5 Gestión de Recambios

**Listado** (`views/recambios/main.php`):
- Selector de vehículo + toggle incluir stock cero
- Cards con icono del grupo, nombre, referencia, stock calculado
- Tap abre offcanvas (editar, eliminar, comprar, ver compras)
- FAB flotante

**Formulario** (`views/recambios/recambio.php`):
- Subida de imagen (mismo patrón upload-container), grupo, referencia, nombre, observaciones

### 8.6 Gestión de Mantenimientos

**Vista** (`views/mantenimientos/mantenimiento.php`):
- 3 pestañas:
  1. Formulario: fecha, km, operación, recambio, localización, grupo, precio, unidades
  2. Observaciones
  3. Adjuntos: upload-container único (cámara/galería), lista de adjuntos subidos
- Cards con swipe para eliminar (Hammer.js)

**Detalle vehículo** (`views/detalles/detalle.php`):
- 2 pestañas con selector de grupo:
  1. **Resumen km** (`getKmsByGrupo`): por localización, muestra últimos km, km realizados, tiempo transcurrido
  2. **Histórico** (`getHistorico`): agrupado por localización con cards de color. Cada registro muestra: fecha, km, operación (icono+nombre), recambio (miniatura+nombre+referencia), precio, unidades, duración km, duración tiempo, edad vehículo, observaciones. Footer con recorrido total.

### 8.7 Gestión de Compras

- Formulario: fecha, proveedor, precio, unidades
- Listado asociado a un recambio

### 8.8 Dashboard Principal

- `views/main.php`: cards de vehículos con acceso directo a mantenimientos
- Sidebar con entrada de km manuales

## 9. Subida de Imágenes

### Controlador (`controllers/attach.php`)

| source | Directorio | Persistencia BD |
|--------|-----------|-----------------|
| vehiculo | assets/images/Vehiculos/ | No (vehiculo.imagen) |
| recambio | assets/images/Recambios/ | No (recambio.imagen) |
| adjunto | attachments/ | Sí (adjuntos) |

### Compresión (`compressImage()`)
- Redimensiona a máx. 1920px lado mayor
- Comprime iterativamente JPEG/PNG/WebP hasta ≤200KB
- Calidad inicial 90, decrementos de 10 hasta 30
- Si aún excede, escala al 80%→30% con calidad 70
- Nombre archivo: UUID + .jpg

### UI Compartida
Los tres formularios con subida de imagen (`vehiculos/form.php`, `recambios/recambio.php`, `mantenimientos/mantenimiento.php`) usan las mismas clases CSS compartidas:
- `.upload-container` - contenedor dashed clickable (centrado, max 260px, 150px alto)
- `.upload-preview` - imagen previa con object-fit: cover
- `.upload-placeholder` - icono cámara FontAwesome + texto

Variables en `theme.css`: `--upload-border`, `--upload-placeholder-text`, `--upload-bg`.

## 10. Sidebar y Navegación

### Menú Lateral (`views/components/sidebar.php`)
- Input de km manuales + botones check/cancel
- Items: Inicio, Mantenimientos, Rutas, Vehículos, Grupos, Recambios, Stock
- Toggle de tema (luna/sol)
- Salir

### Routing (`services/components/sitebar.js`)
`menuAction(action, deep)` calcula basePath según `navigation_deep`:
- deep=0 → `.` (index.php)
- deep=1 → `..` (views/*.php)
- deep=2 → `../..` (views/subcarpeta/*.php)

Casos implementados: inicio, mantenimiento, vehiculos, **grupos**, stock, rutas, recambios, salir.

### Margins en modo oscuro
`.menu-item` con `padding: 2px 6px` para iconos más juntos verticalmente.

## 11. FAB (Floating Action Button)

Presente en listados de vehículos, recambios y grupos. Mismo estilo:
- Posición fija abajo-derecha (bottom: 80px, right: 20px)
- 56x56px, borderRadius 50%
- Background: `--fab-bg` (gradiente corporativo)
  - Claro: `linear-gradient(135deg, #667eea, #764ba2)`
  - Oscuro: `linear-gradient(135deg, #302b63, #24243e)`
- Icono: add.png con `filter: brightness(0) invert(1)` (blanco)
- Efecto active: scale(0.92)

## 12. Histórico Detallado (Detalles Tab2)

### Query (`historico_mantenimientos_by_grupo` en `repositories/mantenimiento.php`)
- CTE `Numeracion` con ventanas `LEAD(kms)` y `LEAD(fecha)` para calcular duración
- JOIN a operaciones, recambios, localizaciones para obtener nombres e imágenes
- Campos: id, fecha, operacion_nombre, operacion_imagen, recambio, recambio_referencia, recambio_imagen, kms, precio, unidades, observaciones, localizacion, localizacion_imagen, duracion_kms, duracion_tiempo (días entre mantenimientos), edad_vehiculo (años+meses desde fecha_compra)

### Render (`parseHtmlCardHistorico` en `services/detalles/detalle.js`)
- Agrupa por localización_id con cabeceras de colores
- Cada item: badge #fila_num, fecha, kms badge, edad vehículo, operación (icono+nombre), recambio (miniatura+nombre+referencia), duración km, periodo, precio, unidades, observaciones
- Footer: recorrido total en km

## 13. Tareas Programadas

### `jobs/cron_email.php`
Script para envío de emails automatizados mediante SMTP Gmail.
```bash
0 * * * * php /var/www/html/gesBike/jobs/cron_email.php
```

## 14. Desarrollo y Contribución

### Estructura de un Nuevo Módulo

Para agregar una nueva entidad:

1. **Crear tabla** en base de datos SQLite
2. **Crear repositorio** en `repositories/{entidad}.php`
3. **Crear modelo** en `models/{entidad}.php`
4. **Crear controlador** en `controllers/{entidad}.php`
5. **Crear API** en `api/{entidad}/{entidad}.php`
6. **Crear vista listado** en `views/{entidad}/main.php`
7. **Crear vista formulario** en `views/{entidad}/form.php`
8. **Crear servicio JS** en `services/{entidad}/{entidad}.js`
9. **Crear CSS** en `assets/css/{entidad}/{entidad}.css`
10. **Añadir caso en sitebar.js** para routing
11. **Añadir theme.css** enlace en el `<head>` de la vista
12. **Añadir theme.js** y `initTheme()` en el `onload`

### Convenciones

- **snake_case** para nombres de archivos PHP y funciones
- **camelCase** para funciones y variables JavaScript
- Tablas con **is_active** para borrado lógico
- Timestamps en **created_at**, **modified_at**, **deleted_at**
- IDs autoincrementales como clave primaria
- **CSS variables** para colores (nunca valores fijos), definidas en `theme.css`
- Transiciones suaves en cambios de color (`transition: xxx 0.3s ease`)
- Fecha ISO en inputs tipo date: `YYYY-MM-DD`

## 15. Notas Adicionales

- **Borrado lógico** (soft delete) mediante `is_active=0` + `deleted_at`
- **Imágenes vehículos**: `assets/images/Vehiculos/` con nombre UUID
- **Imágenes recambios**: `assets/images/Recambios/` con nombre UUID
- **Iconos grupos**: 22 PNGs predefinidos en `assets/images/icons/Grupos/`
- **Categoría vehículo**: solo dos valores: `pulmonar` o `electrica`
- **CSS compartido**: `.upload-container/upload-preview/upload-placeholder` en `style.css` para subida de imágenes
- **Fondo páginas**: gradiente púrpura suave `#f0ecf9→#f8f4ff` en modo claro, sólido `#1a1a2e` en oscuro
- **Login**: gradiente púrpura intenso `#667eea→#764ba2` (claro), `#0f0c29→#302b63→#24243e` (oscuro)
- **Autologin**: detecta credenciales precargadas y ejecuta auth en intervalos de 500ms, touchstart y visibilitychange
- **Histórico km**: query con ventanas `LEAD()` para calcular duración de recambios entre mantenimientos
- **Totalizador km**: suma de `kms_actuales` desde `ultimos_kms` con fallback a `kms_inicio`

---
> Source: [calcena/gesbike](https://github.com/calcena/gesbike) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
