## portfolio2026-convenciones-del-proyecto

> - Angular 22 standalone zoneless — nunca usar NgModules


# Portfolio2026 — Convenciones del proyecto

## Stack

- Angular 22 standalone zoneless — nunca usar NgModules
- NX monorepo — estructura `apps/portfolio` (frontend), `apps/lab001` (remote de Native Federation) y `apps/backend` (backend, proyecto npm independiente con su propio package.json/lockfile)
- Native Federation con `@angular-architects/native-federation` 22.0.6
- SCSS con alias `v` para variables, `m` para mixins
- BEM, rem para espaciados, px para bordes
- Signals y inject() — nunca constructor injection
- Path aliases: @core, @shared, @features
- SSR activo en producción — desactivado en desarrollo
- Node 22 requerido — gestionado con Volta
- TypeScript fijado a `~6.0.0` en todo el workspace (requerido por `build-angular@22`) — no actualizar sin revisar antes si rompe `ignoreDeprecations` o la inferencia de `rootDir` en `apps/backend`

## Arquitectura

- Smart/Dumb components
- SSR activo en producción — compatibilidad servidor/cliente obligatoria
- Angular standalone y signals en componentes
- Nunca inyectar servicios en constructor, usar `inject()`
- Servicios anotados con `@Service()` — decorador ya aplicado a todos los servicios del proyecto
- Peticiones HTTP reactivas (queries) a través de `httpResource`, no `HttpClient` manual con Observables — excepción: acciones puntuales que no son queries reactivas (ej. `admin.guard.ts`, `ai.service.ts`) pueden seguir en `HttpClient`/Observable
- Formularios con Signal Forms + Submission API — no Reactive Forms ni Template-driven

## GSAP

- No modificar lógica GSAP existente sin indicación explícita
- ngOnDestroy obligatorio en componentes con ScrollTrigger
- animationInitialized flag en componentes con animaciones
- Los ScrollTrigger que se guarden en el componente deben ser signals (`WritableSignal<ScrollTrigger[]>`), nunca propiedades planas — afecta también a cómo se testean (ver Tests)

## Pipes y localización

- Los pipes que dependan de locales de Angular deben evitarse — hay un bug conocido con `LOCALE_ID` + `isolatedModules`
- Usar `Intl` nativo del browser como alternativa: crear pipes personalizados con `Intl.DateTimeFormat`, `Intl.NumberFormat`, etc. según el caso

## Tests

- Jest — 0 fallos obligatorio
- Mantener cobertura existente en cualquier cambio
- Mover los mocks de frontend a archivos individuales bajo `src/app/core/mocks/`
- Cada spec debe importar el mock específico desde `@core/mocks/...` en lugar de definir mocks inline cuando sea posible
- Tests de servicios/componentes que usan `httpResource`: el `TestBed` necesita `provideHttpClient()` + `provideHttpClientTesting()` (o mockear el servicio entero con signals planas si el componente no necesita el HTTP real). El `httpResource` resuelve su respuesta vía microtask — tras `req.flush(...)` hace falta `await Promise.resolve()` (test `async`) antes del `TestBed.tick()` final, o el estado se queda en `loading`
- Signals internas de componente (ej. `scrollTriggers`, `lastAnimatedCount`) nunca se sobreescriben por asignación directa en los specs (`component['x'] = valor`) — siempre `.set(valor)` para escribir y `component['x']()` para leer
- Backend (NestJS): `@typescript-eslint/unbound-method` da falso positivo con `expect(mock.metodo).toHaveBeenCalledWith(...)` — está desactivado vía override en `eslint.config.mjs` solo para `**/*.spec.ts`/`test/**/*.ts`, no es necesario evitar ese patrón de test
- Backend: `apps/backend/tsconfig.json` (base) debe incluir los `*.spec.ts` para que ESLint los reconozca — la exclusión de specs para el build vive en `apps/backend/tsconfig.build.json`, no en el tsconfig base

## Pages y SEO

- Al crear un componente `page`, además de registrar la ruta, revisar siempre el archivo `ts` y aplicar el SEO correspondiente
- En un componente `page` se debe inyectar `SeoService` con `inject(SeoService)` y definir `ngOnInit()`
- Dentro de `ngOnInit()` debe llamarse:
  - `this.seoService.update(PAGE_SEO['<page>']);`
  - `this.seoService.updateSchemas([BREADCRUMB_<PAGE>]);` cuando exista el breadcrumb
- Si una página no tiene breadcrumb definido, al menos debe actualizar su SEO con `PAGE_SEO['<page>']`
- Si la página es nueva, añadirla también al sitemap.xml

## SSR

- En desarrollo SSR está desactivado — `initNodeFederation` solo se ejecuta en producción (`NODE_ENV === 'production'`)
- `provideClientHydration` solo en producción — en desarrollo usar array vacío para evitar errores de hidratación
- Código que accede a `window`, `document` o `localStorage` debe protegerse siempre con `PlatformService.isBrowser`
- Bloques `@defer` con contenido que anima al hacer scroll deben usar `hydrate on viewport` — contenido visible en el primer viewport (`app-hero`, `app-skills`) queda deliberadamente fuera de `@defer`

## Native Federation

- `federation.config.js` en `apps/portfolio/`
- Los path aliases internos (`@core`, `@shared`, `@features`) deben estar en el array `skip` — no son libs federadas
- `initFederation()` en `main.ts` solo se ejecuta en browser — guard con `typeof window !== 'undefined'`

## CI/CD

- Pipeline en GitHub Actions — se ejecuta en push a `develop` y `feature/**`
- Jobs: lint → test → build-backend → deploy-backend → wait-for-backend → build-frontend → deploy-frontend
- Deploy solo en rama `develop`
- Node 22 en todos los jobs
- Frontend desplegado en rama `deploy`, backend en rama `deploy-api`

## Backend

- NestJS en `apps/backend/`
- API: https://api.csrangulardeveloper.nom.es/api/v1
- Evitar `any` en tipos que representan datos con forma variable (ej. configuración de una stat) — usar uniones discriminadas por una propiedad tipo `kind`/`type` en vez de `any`, y `unknown` en vez de `any` para valores genuinamente dinámicos (ej. inputs de componentes cargados dinámicamente)

## Idioma

- Responder siempre en castellano

---
> Source: [SA-full-stack-developer/portfolio2026_A](https://github.com/SA-full-stack-developer/portfolio2026_A) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
