## zuzki-dev-front

> Este archivo proporciona el contexto necesario para que Gemini (u otros agentes de IA) entiendan la estructura, tecnologías y convenciones de este proyecto.

# Contexto del Proyecto: Zuzki Dev Front

Este archivo proporciona el contexto necesario para que Gemini (u otros agentes de IA) entiendan la estructura, tecnologías y convenciones de este proyecto.

## 🚀 Resumen del Proyecto
**Zuzki Dev Front** es una plataforma web moderna que funciona como portafolio personal y panel de administración. Está diseñado para ser extremadamente rápido, accesible y optimizado para despliegues en el "Edge" (Cloudflare Pages).

- **Tecnología Principal:** Angular 21.0.1 (Zoneless, Signals, Standalone).
- **Estilos:** Tailwind CSS 4.1.14 (PostCSS).
- **Infraestructura:** Cloudflare Pages + Wrangler.
- **Arquitectura:** Core-Centric (Simplificada).

## 🏗️ Arquitectura y Estructura
El proyecto ha evolucionado desde una arquitectura hexagonal hacia un modelo más directo centrado en el `Core`.

### Capas Principales
1.  **Core (`src/app/core/`)**:
    *   **Stores**: Servicios `@Injectable` que gestionan el estado reactivo mediante Signals (`WritableSignal`).
    *   **Services**: Comunicación directa con APIs externas (GitHub, Custom Backend).
    *   **Interceptors**: Interceptores funcionales para autenticación y selección dinámica de API.
    *   **Interfaces**: Definiciones de tipos y contratos de datos.
2.  **Features (`src/app/features/`)**: Módulos funcionales (admin, auth, home, works) que contienen la lógica de negocio visual.
3.  **Pages (`src/app/pages/`)**: Componentes de alto nivel vinculados a las rutas que orquestan las features.
4.  **Shared (`src/app/shared/`)**: UI Atómica, modales y utilidades reutilizables.

## 📚 Documentación de Referencia
Para obtener detalles específicos sobre cómo implementar tareas, consulta las siguientes guías:

- **Índice de Documentación:** [`docs/README.md`](docs/README.md)
- **Arquitectura Detallada:** [`docs/architecture-core-centric.md`](docs/architecture-core-centric.md)
- **Estándares de Angular:** [`docs/standards/angular.md`](docs/standards/angular.md)
- **Diseño y Tokens:** [`docs/standards/design-tokens.md`](docs/standards/design-tokens.md)
- **Componentes de UI:** [`docs/standards/ui-components.md`](docs/standards/ui-components.md)
- **Accesibilidad:** [`docs/standards/accessibility.md`](docs/standards/accessibility.md)
- **API Backend:** [`docs/backend-documentation.md`](docs/backend-documentation.md)

## 🛠️ Comandos Clave (pnpm)
- `pnpm start`: Ejecuta el build y arranca el entorno de desarrollo local con `wrangler`.
- `pnpm run build`: Genera el build de producción y ejecuta el post-procesamiento.
- `pnpm run deploy`: Construye y despliega directamente a Cloudflare Pages.
- `pnpm run lint`: Ejecuta ESLint para asegurar la calidad del código.
- `pnpm run test`: Ejecuta los tests unitarios con Karma/Jasmine.
- `pnpm run test:a11y`: Ejecuta pruebas de accesibilidad automáticas.

## 📏 Convenciones de Desarrollo
- **Standalone:** Todos los componentes, directivas y pipes **deben** ser `standalone: true`.
- **Inyección de Dependencias:** Usar la función `inject()` en lugar del constructor siempre que sea posible.
- **Signals:** Obligatorio para la gestión de estado y reactividad. No usar `Zone.js` (Zoneless activado).
- **Obtención de Datos:** Usar `rxResource` para una integración nativa con SSR y TransferState.
- **Caché:** Se utiliza un interceptor de caché HTTP global para optimizar la navegación y reducir la carga del servidor.
- **SSR Crítico:** Evitar el uso de `@defer` en secciones fundamentales para el SEO (Proyectos, Blog) para garantizar su renderizado en el servidor.
- **Control Flow:** Usar la nueva sintaxis de Angular (`@if`, `@for`, `@switch`).
- **Estilos:** Seguir el sistema de diseño definido en `src/styles/` y usar clases de Tailwind CSS.
- **Routing:** Las rutas usan `loadComponent` para Lazy Loading por defecto.

## 📝 Notas de Infraestructura
- El proyecto utiliza un paso de post-procesamiento (`pnpm run process`) que ejecuta herramientas en `tools/` para copiar archivos de configuración de Cloudflare y generar el `sitemap.xml`.
- Las variables de entorno se gestionan en `src/environments/`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sleepy-zuzki) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
