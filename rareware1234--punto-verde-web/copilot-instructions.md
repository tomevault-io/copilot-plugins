## punto-verde-web

> Sistema POS web para Punto Verde — tiendas de abarrotes. Frontend React de la misma app que la versión Swift/macOS. Comparten el mismo Firestore.

# punto-verde-web

Sistema POS web para Punto Verde — tiendas de abarrotes. Frontend React de la misma app que la versión Swift/macOS. Comparten el mismo Firestore.

## Firebase

| Campo | Valor |
|-------|-------|
| Proyecto | `punto-verde-7b743` |
| Auth | Email/Password |
| Firestore | Producción |
| Hosting | Firebase Hosting → `dist/` |
| Config | `src/firebase-config/firebase-config.js` |
| Singleton | `src/firebase.js` (guarded con `getApps()`) |

> **TODO**: `appId` en firebase-config.js es placeholder `"1:735728617383:web:pending"`.
> Reemplazar con el appId web real desde Firebase Console → punto-verde-7b743 → Project Settings → Your Apps → Web.
> Funciona sin él pero es técnicamente incompleto.

## Stack

- React 19 + Vite 8
- Firebase JS SDK v11 (Auth + Firestore + Storage)
- React Router v7 — **HashRouter** (rutas con `#`)
- Bootstrap 5 (solo clases, sin componentes JS)
- Chart.js + react-chartjs-2
- react-icons, @zxing/browser (scanner de código de barras)

## Autenticación

El login usa `numEmpleado`, NO email directamente:

```
numEmpleado → query Firestore(empleados) → obtener email real → signInWithEmailAndPassword
```

- Colección Firestore: `empleados`
- Campos relevantes: `numEmpleado`, `email`, `nombre`, `rol`, `tiendaId`, `tiendasAsignadas`, `activo`
- Sesión en `sessionStorage` bajo clave `desktop_empleado` (JSON)
- Firebase Auth persiste entre recargas; sessionStorage se verifica en cada `onAuthStateChanged`

## Roles y permisos

| Firestore (`rol`) | Rol interno | Tema CSS |
|-------------------|-------------|----------|
| STAFF | `staff` | Verde `#1A7A48` |
| LIDER | `manager` | Azul `#2E6BC4` |
| MANAGER | `manager` | Azul `#2E6BC4` |
| DIRECTOR | `admin` | Gris `#64748B` + ámbar |
| ADMIN | `admin` | Gris `#64748B` + ámbar |

Permisos disponibles (ver `AuthContext.jsx` → `PERMISOS`):
`ventas`, `productos_ver/editar/agregar/eliminar`, `caja_consulta/operar`,
`reportes`, `empleados_ver/editar/crear`, `tiendas_ver/editar/crear`,
`turnos_ver/editar/crear`, `tareas_ver/editar/crear`,
`promociones_ver/editar/crear`, `creditos_ver/aprobar/editar/suspender`, `configuracion`

Los temas de rol se aplican como CSS variables en `:root` (ver `aplicarTemaRol` en AuthContext).

## Rutas

| Ruta | Página | Permiso requerido |
|------|--------|-------------------|
| `/` | Venta (POS) | ventas |
| `/caja` | Caja | — |
| `/productos` | Productos | productos_ver |
| `/pedidos` | Pedidos | — |
| `/tareas` | Tareas | — |
| `/empleados` | Empleados | empleados_ver |
| `/tiendas` | Sucursales | tiendas_ver |
| `/turnos` | Horarios | turnos_ver |
| `/creditos` | Crédito | creditos_aprobar |
| `/dashboard` | Dashboard | reportes |
| `/promociones` | Promociones | promociones_ver |
| `/perfil` | Mi Perfil | ventas |
| `/configuracion` | Configuración | configuracion |
| `/pantalla-cliente` | Pantalla cliente | público |
| `/login` | Login | público |

## Estructura

```
src/
├── firebase-config/   ← configuración de Firebase
├── firebase.js        ← singleton de app/db/auth/storage
├── context/           ← AuthContext, CartContext
├── pages/             ← una página por ruta
├── components/        ← componentes reutilizables
│   ├── empleado/      ← widgets del perfil/dashboard empleado
│   └── creditos/      ← componentes del módulo de crédito
├── services/          ← acceso a Firestore (un archivo por entidad)
├── hooks/             ← hooks personalizados
├── data/              ← datos estáticos (employeeProfiles, products.json)
├── assets/            ← imágenes y logos
└── styles/            ← CSS adicionales por módulo
```

## Correr en desarrollo

```bash
npm run dev   # Vite en http://localhost:5173
```

## Desplegar

```bash
npm run build
firebase deploy --only hosting
```

El `.firebaserc` apunta a `punto-verde-7b743`.

## Notas importantes

- **Vite HMR no reinicializa Firebase**: Si cambias `firebase-config.js`, haz full reload (`Cmd+Shift+R`), no basta con guardar.
- **HashRouter**: Las rutas son `/#/productos`, no `/productos`. No confundir al revisar URLs.
- **Sin appId web real**: El placeholder funciona para Auth y Firestore, pero para Analytics o funciones que validan platform type, se necesita el appId correcto.
- `src/data/products.json` — datos de prueba locales. En producción los productos vienen de Firestore via `productService.js`.

---
> Source: [rareware1234/punto-verde-web](https://github.com/rareware1234/punto-verde-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
