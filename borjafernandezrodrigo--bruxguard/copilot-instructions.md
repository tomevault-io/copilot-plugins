## bruxguard

> BruxGuard es una app móvil de uso personal, offline-first, para la concienciación sobre el bruxismo (apretamiento/rechinamiento de dientes). Utiliza notificaciones locales adaptativas para construir consciencia corporal durante el día y registrar patrones a lo largo del tiempo.

# BruxGuard — Contexto para Agentes IA

## Descripción del Proyecto

BruxGuard es una app móvil de uso personal, offline-first, para la concienciación sobre el bruxismo (apretamiento/rechinamiento de dientes). Utiliza notificaciones locales adaptativas para construir consciencia corporal durante el día y registrar patrones a lo largo del tiempo.

**Sin backend. Sin autenticación. Sin sincronización en la nube. 100% local.**

---

## Stack Tecnológico

- **Framework móvil:** Ionic + Angular (Ionic Angular)
- **Runtime / Native bridges:** Capacitor
- **Runtime alternativo (opcional):** Expo (managed workflow) — usaremos `expo-sqlite` para acceso SQLite en entornos Expo
- **Persistencia local:** SQLite (via Capacitor community plugin o `expo-sqlite` según runtime)
- **Testing:** Jasmine + Karma (unit tests)
- **Linting & Formato:** ESLint, Prettier
- **Lenguaje:** TypeScript

## Estructura del Proyecto

```
bruxguard/
├── .opencode/           # Configuración y código de agentes IA
│   ├── agents/
│   │   └── ... (agentes)
│   ├── skills/
│   │   ├── ... (skills de agentes)
│   └── prompts/
│       └── ... (prompts)
├── src/
│   ├── app/
|   |── shared/       # Código compartido entre capas (DTOs, interfaces, utilidades)
|   │   │   ├── domain/
|   |   │   │   ├── value-objects/ # Objetos de valor root del dominio
|   |   │   │   ├── types/         # Tipos TypeScript globales
|   |   │   │   ├── services/      # Lógica de negocio que no encaja en entidades o casos de uso
|   │   │   ├── infrastructure/ # conectores y adaptadores reales a servicios externos o APIs (ej: almacenamiento local, notificaciones)
|   │── modules/     # Funcionalidades organizadas por módulos (ej: registro-sesion, historial, ajustes)
|   │   │   ├── nombre-modulo/
|   │   │   │   ├── domain/       # Lógica de negocio, entidades, casos de uso
|   │   │   |   |   ├── value-objects/ # Objetos de valor, tipos y validaciones
|   │   │   |   |   ├── types/         # Tipos TypeScript globales
|   │   │   |   |   ├── aggregates/    # Agregados de dominio
|   │   │   |   |   ├── services/      # Lógica de negocio que no encaja en entidades o casos de uso
|   │   │   │   ├── infrastructure/ # Acceso a datos, repositorios
|   |   |   |   |   ├── ui/             # Componentes, páginas, estilos
|   |   |   |   |   |    |── components
|   |   |   |   |   |    |    |── <nombre del componente>.html
|   |   |   |   |   |    |    |── <nombre del componente>.scss
|   |   |   |   |   |    |    |── <nombre del componente>.ts
|   |   |   |   |   |    |── pages
|   |   |   |   |   │   ├── domain/    # Lógica de negocio específica del módulo
|   |   |   |   |   ├── repository
|   |   |   |   |   |    |── <nombre del repositorio>.repository.ts # implementación concreta del repositorio (adaptador arquitectura hexagonal)
|   |   |   |   |   |    |── <nombre del repositorio>.infrastructure.ts # interfacce del repositorio (port arquitectura hexagonal)
|   |   │   │   ├── application/  # Coordinación de casos de uso
│   ├── assets/
│   ├── enviroments/ # Configuración de entornos (dev, staging, prod)
├── tests/             # Tests unitarios, de integración y end-to-end tiene la misma estructura que `app/` para facilitar la localización de pruebas relacionadas con cada módulo o capa.
```

## Agentes IA

### Agentes Primarios

| Agente | Tipo | Modelo | Temp | Invocación |
|--------|------|--------|------|------------|
| `plan` | primary | `opencode/qwen3.6-plus-free` | 0.1 | Tab |
| `build` | primary | `opencode/nemotron-3-super-free` | 0.3 | Tab |

#### `plan` — Planificador y Arquitecto

Analiza, diseña y planifica. **Nunca implementa**. Produce hitos y tareas antes de cualquier cambio de código.

- `write`: ✓ (genera ficheros de plan) · `edit`: ✗ · `bash`: ✗
- Skills autorizados: `generador-hito`, `generador-tarea`
- Puede delegar a: `@documentador`, `@explore`
- Prompt: `.opencode/prompts/plan.md`

#### `build` — Constructor

Implementa las tareas producidas por `plan`. Acceso completo con confirmación en escritura y bash.

- `write`: ✓ · `edit`: ask · `bash`: ask
- Prompt: agente integrado de OpenCode

---

### Subagentes

| Agente | Modelo | Temp | Invocación | Escribe |
|--------|--------|------|------------|---------|
| `documentador` | `opencode/minimax-m2.5-free` | 0.1 | `@documentador` | ✓ |

#### `documentador` — Documentador Técnico

Documenta el estado real del proyecto y las decisiones técnicas. Mantiene `AGENTS.md`, `README.md`, ADRs y JSDoc.

- `write`: ✓ · `edit`: allow · `bash`: ✗ · `webfetch`: ✗
- Skills autorizados: `auditor-doc-codigo`, `filtro-rigor-tecnico`, `generador-flujo-mermaid`, `agregador-adr`, `arquitectura-agentes`, `generador-jsdoc`
- Configuración: `.opencode/agents/documentador.md`

---

### Skills Disponibles

| Skill | Responsable | Propósito |
|-------|-------------|-----------|
| `generador-hito` | `plan` | Genera `.opencode/plan/<slug>/hito.md` |
| `generador-tarea` | `plan` | Genera `.opencode/plan/<slug>/<tarea>.issue.md` |
| `generador-jsdoc` | `documentador` | Bloques JSDoc para TypeScript/JavaScript |
| `auditor-doc-codigo` | `documentador` | Detecta discrepancias entre JSDoc e implementación |
| `filtro-rigor-tecnico` | `documentador` | Elimina lenguaje impreciso en documentación |
| `generador-flujo-mermaid` | `documentador` | Diagramas Mermaid para secuencias y flujos |
| `agregador-adr` | `documentador` | Borradores de ADR para decisiones arquitectónicas |
| `arquitectura-agentes` | `documentador` | Mantiene actualizado el mapa de agentes |
| `comentarista-logica-interna` | `build` | Documenta lógica interna, hacks y deuda técnica |
| `skill-generator` | cualquiera | Scaffold para crear nuevos skills |

## Comandos de Desarrollo

| Comando | Descripción |
|---------|-------------|
| `npm run agent:plan` | Inicia sesión de planificación con `plan` |
| `npm run agent:build` | Inicia sesión de construcción con `build` |
| `npm start` | Dev server en `localhost:4200` |
| `npm run build` | Compila para producción (output: `www/`) |
| `npm run watch` | Build en modo observación (dev) |
| `npm test` | Ejecuta todos los tests con Karma + Jasmine |
| `npm test -- --include=src/app/home/home.page.spec.ts` | Ejecuta un solo test |
| `npm test -- --watch=false --browsers=ChromeHeadless` | Tests CI (sin watch, headless) |
| `npm run lint` | ESLint sobre `src/**/*.ts` y `src/**/*.html` |
| `npm run validate-skills` | Valida skills de agentes IA |

## Convenciones de Código

### TypeScript
- **Strict mode** activado (`strict: true` en `tsconfig.json`)
- `noImplicitOverride`, `noImplicitReturns`, `noFallthroughCasesInSwitch`, `noPropertyAccessFromIndexSignature` habilitados
- `experimentalDecorators: true` requerido para Angular
- Usar `standalone: true` en componentes y páginas (scaffold por defecto)

### Imports y Estructura
- Importar componentes Ionic desde `@ionic/angular/standalone` (no desde `@ionic/angular`)
- Orden de imports: Angular → Ionic → terceros → relativos
- Path base: `baseUrl: "./"` en tsconfig (imports relativos desde raíz del proyecto)

### Naming
- **Componentes:** sufijo `Page` o `Component` (regla ESLint `@angular-eslint/component-class-suffix`)
- **Selectores:** prefijo `app` en kebab-case (ej: `app-home`, `app-root`)
- **Directivas:** prefijo `app` en camelCase
- **Archivos:** kebab-case (ej: `home.page.ts`, `mi-servicio.service.ts`)

### Estilos
- Preprocesador: **SCSS** (configurado en `@ionic/angular-toolkit`)
- Límite de estilo por componente: warn 2kb, error 4kb

### Testing
- Framework: **Jasmine** con **Karma** como runner
- Patrón: `describe('NombreClase', () => { ... })` con `beforeEach(async () => { ... })`
- Usar `TestBed.configureTestingModule` con `imports` (standalone) o `declarations` (NgModule)
- Tests co-ubicados con el código bajo prueba (`*.spec.ts`)

### Error Handling
- Lanzar errores con `throw new Error('mensaje')` para casos de negocio
- Usar `Observable` + `catchError` de RxJS para errores asíncronos
- No silenciar errores sin logging o recuperación explícita

## Lo que NO se Debe Hacer

- NO escribir código de implementación en `plan`. Solo análisis y diseño.
- NO modificar `AGENT.md` sin confirmación explícita.
- NO asumir permisos o responsabilidades de agentes sin evidencia clara.
- NO ejecutar scripts remotos ni modificar código de agentes sin autorización explícita.
- NO generar o modificar documentación sin usar los skills autorizados para ello.
- NO crear o eliminar agentes automáticamente. Cualquier cambio en la estructura de agentes debe ser aprobado y realizado manualmente por un desarrollador.
- NO sobreescribir archivos de planificación (`hito.md`, `issue.md`) sin autorizacion del usuario mediante `force: true` en la entrada del skill correspondiente.
- NO asignar tareas a subagentes de forma genérica (`"*"`). El subagente responsable debe ser explícito para garantizar trazabilidad y responsabilidad clara.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BorjaFernandezRodrigo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BorjaFernandezRodrigo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
