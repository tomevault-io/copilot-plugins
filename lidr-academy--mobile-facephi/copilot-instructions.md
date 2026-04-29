## mobile-facephi

> > **Última actualización:** 2026-02-15

# AGENTS.md — Sistema de Agentes para Desarrollo Móvil

> **Versión:** 2.0.0
> **Última actualización:** 2026-02-15
> **Propósito:** Definir roles, contratos, inputs/outputs y hand-offs para desarrollo de features móviles con agentes de IA.
> **Regla:** Si hay conflicto entre este archivo y cualquier otro, AGENTS.md prevalece.

---

## 1. Visión General del Sistema

Este sistema define un workflow repetible para construir features móviles (Android + iOS) desde un ticket hasta un PR listo para merge, asistido por agentes de IA.

### Principios

- **Contratos por etapa:** Cada agente produce artefactos con formato estándar
- **Gates explícitos:** No se avanza sin cumplir criterios de calidad
- **Herramienta-agnóstico:** Los roles no cambian si cambias Claude Code por Cursor, Copilot u otra herramienta
- **Spec Atómica:** Cada especificación debe ser autocontenida y ejecutable en un solo ciclo por el agente
- **Seguridad integrada:** OWASP MASVS se valida en cada etapa, no al final

### Estructura del Repositorio

```
AGENTS.md                          ← Este archivo (source of truth)
CLAUDE.md                          ← Contexto del proyecto para el agente
.claude/
  skills/
    owasp-mobile/SKILL.md          ← Revisión de seguridad OWASP MASVS
    api-consume/SKILL.md           ← Consumo seguro de APIs
    arch-check/SKILL.md            ← Guardia de Clean Architecture
    figma-to-ui/SKILL.md           ← Diseño a código nativo
  commands/
    spec.md                        ← /spec - Genera spec atómica
    api.md                         ← /api - Consume endpoint
    security.md                    ← /security - Auditoría OWASP
    ui.md                          ← /ui - Diseño a código
    arch.md                        ← /arch - Validar arquitectura
    pr.md                          ← /pr - Preparar PR
docs/
  features/
    FEAT-XXX/                      ← Una carpeta por feature
      spec.md
      ui_contract.md
      arch.md
      tasks.md
      qa_plan.md
      risks.md
      pr.md
  decisions.md                     ← ADRs (Architecture Decision Records)
rules/
  engineering.md                   ← Convenciones generales
  mobile_android.md                ← Reglas específicas Android
  mobile_ios.md                    ← Reglas específicas iOS
  security_privacy.md              ← Seguridad y privacidad
```

---

## 2. Flujo General

```
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 0: Bootstrap (1 vez por proyecto)                    │
│  → Configurar CLAUDE.md + AGENTS.md + /skills + /rules     │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 1: Spec Agent                                        │
│  Ticket → spec.md + risks.md + tasks.md + qa_plan.md        │
│  Gate: ACs testables + 4 estados UI definidos               │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 2: Design Translator                                 │
│  Figma/screenshot + spec.md → ui_contract.md                │
│  Gate: State machine completa + tokens Design System        │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 3: Mobile Architect                                  │
│  spec.md + ui_contract.md → arch.md + tasks.md              │
│  Gate: Sigue Clean Architecture + dependencias validadas    │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 4: Code Agent (por vertical slice)                   │
│  arch.md + tasks.md → Código implementado                   │
│  Gate: Lint pass + tests pass + /security pass              │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  ETAPA 5: PR Guardian                                       │
│  Código + spec + arch → PR listo para merge                 │
│  Gate: DoD completo + OWASP check + tests verdes            │
└─────────────────────────────────────────────────────────────┘
```

---

## 3. Definición de Agentes

---

### 3.1 Spec Agent

**Responsabilidad:** Transformar un ticket ambiguo en una especificación verificable y atómica.

**Cuándo se activa:** Al iniciar cualquier feature nueva o cambio significativo.

**Inputs requeridos:**

| Input | Fuente | Obligatorio |
|-------|--------|:-----------:|
| Ticket URL o contenido | Jira / Linear / GitHub Issues | ✅ |
| Contexto de negocio | PM o documentación del producto | ✅ |
| Diseños (si existen) | Figma URL o screenshot | ⚠️ Opcional |
| API specs (si existen) | Swagger / OpenAPI | ⚠️ Opcional |

**Outputs generados:**

Todos los archivos se crean en `docs/features/FEAT-XXX/`:

- **spec.md** — Especificación completa con:
  - Resumen del feature (1-2 párrafos)
  - User Stories en formato "Como [rol], quiero [acción], para [beneficio]"
  - Acceptance Criteria en formato Given/When/Then
  - Estados de UI: Loading, Content, Error, Empty
  - Eventos de analytics (nombre, trigger, propiedades)
  - Feature flags definidos (si aplica rollout gradual)

- **risks.md** — Riesgos identificados con mitigación:
  - Técnicos (APIs lentas, complejidad, dependencias)
  - De negocio (requisitos cambiantes, prioridades)
  - De timeline (dependencias bloqueantes)

- **tasks.md** — Desglose en tareas por vertical slice:
  - Slice 1: Happy path + loading
  - Slice 2: Empty + error + retry
  - Slice 3: Analytics + feature flag + a11y + i18n

- **qa_plan.md** — Plan de QA:
  - Happy paths
  - Edge cases
  - Error scenarios
  - Dispositivos target

**Gates de validación (todos deben cumplirse):**

- [ ] Todos los AC son testables (Given/When/Then)
- [ ] Los 4 estados de UI están definidos (Loading/Content/Error/Empty)
- [ ] Analytics events tienen nombres consistentes con el estándar del proyecto
- [ ] Feature flags están definidos si es rollout gradual
- [ ] Dependencias tienen estado (disponible/bloqueado)
- [ ] Riesgos tienen mitigación propuesta
- [ ] QA plan cubre happy path + edge cases + errors

**Hand-off:** → Design Translator

**Comando asociado:** `/spec FEAT-XXX`

---

### 3.2 Design Translator

**Responsabilidad:** Convertir un diseño de Figma (via MCP o screenshot) en un contrato de UI ejecutable, mapeando al Design System del proyecto.

**Cuándo se activa:** Cuando existe diseño para un feature y el spec.md está aprobado.

**Inputs requeridos:**

| Input | Fuente | Obligatorio |
|-------|--------|:-----------:|
| Diseño | Figma MCP / screenshot (Ctrl+V) / URL | ✅ |
| spec.md | docs/features/FEAT-XXX/ | ✅ |
| Design System tokens | Documentación del proyecto | ✅ |

**Outputs generados:**

- **ui_contract.md** — Contrato de UI que incluye:
  - Componentes identificados (nuevos vs reutilizables)
  - Mapeo a tokens del Design System (colores, tipografía, spacing)
  - State machine completa:

| Estado | Android (Composable) | iOS (SwiftUI View) | Trigger |
|--------|---------------------|--------------------|---------|
| Loading | LoadingScreen() | LoadingView | onAppear / init |
| Content | ContentScreen() | ContentView | datos cargados |
| Error | ErrorScreen() | ErrorView | API failure |
| Empty | EmptyScreen() | EmptyView | data.isEmpty |

  - Checklist de accesibilidad:
    - [ ] Content descriptions (Android) / Accessibility labels (iOS)
    - [ ] Dynamic Type / Scaled fonts support
    - [ ] Minimum touch targets (48dp Android / 44pt iOS)
    - [ ] Color contrast ratio ≥ 4.5:1
    - [ ] Screen reader navigation order

  - Platform-specific considerations:
    - Android: Material 3 component mapping
    - iOS: Human Interface Guidelines compliance

**Gates de validación:**

- [ ] Todos los estados de UI tienen representación visual definida
- [ ] Componentes mapeados a Design System tokens existentes
- [ ] Checklist de accesibilidad completado
- [ ] Consideraciones por plataforma documentadas

**Hand-off:** → Mobile Architect

**Comando asociado:** `/ui` (con imagen pegada o MCP Figma activo)

---

### 3.3 Mobile Architect

**Responsabilidad:** Definir la arquitectura técnica del feature y desglosar en tareas implementables, asegurando que sigue los patrones del proyecto.

**Cuándo se activa:** Cuando spec.md y ui_contract.md están aprobados.

**Inputs requeridos:**

| Input | Fuente | Obligatorio |
|-------|--------|:-----------:|
| spec.md | docs/features/FEAT-XXX/ | ✅ |
| ui_contract.md | docs/features/FEAT-XXX/ | ✅ |
| Codebase existente | Repositorio del proyecto | ✅ |
| API contracts | Swagger / OpenAPI / documentación | ⚠️ Si aplica |

**Outputs generados:**

- **arch.md** — Documento de arquitectura:
  - Diagrama de dependencias del feature
  - Estructura de archivos por capa (Clean Architecture):

```
feature-xxx/
  presentation/
    XxxScreen.kt / XxxView.swift
    XxxViewModel.kt / XxxViewModel.swift
    XxxUiState.kt / XxxUiState.swift
  domain/
    model/         → Entidades puras (sin anotaciones de framework)
    usecase/       → Un caso de uso = una responsabilidad
    repository/    → Interface solamente (NO implementación)
  data/
    repository/    → Implementación del repository
    remote/        → DTOs + API service
    local/         → DAOs + entities de Room/CoreData
    mapper/        → DTO ↔ Domain ↔ Entity
```

  - Contratos de datos (sealed class / enum para estados):

```kotlin
// Android
sealed interface UserProfileState {
    object Loading : UserProfileState
    data class Content(val user: User) : UserProfileState
    data class Error(val message: String, val retry: () -> Unit) : UserProfileState
    object Empty : UserProfileState
}
```

```swift
// iOS
enum UserProfileState {
    case loading
    case content(User)
    case error(String, retry: () -> Void)
    case empty
}
```

  - Dependencias (nuevas vs existentes)
  - Decisiones técnicas con justificación (ADR format)

- **tasks.md** (actualizado) — Tareas refinadas por vertical slice:
  - Cada tarea tiene: descripción, archivos a crear/modificar, criterio de done
  - Estimación de complejidad (S/M/L)
  - Orden de ejecución y dependencias entre tareas

**Gates de validación:**

- [ ] Sigue la regla de dependencia: `presentation → domain ← data`
- [ ] domain/ NO importa de presentation/ ni data/
- [ ] ViewModels solo dependen de UseCases
- [ ] UseCases reciben Repository interface (no implementación)
- [ ] No se agregan dependencias nuevas sin justificación en arch.md
- [ ] Feature flags definidos
- [ ] Eventos de analytics tienen implementación propuesta

**Hand-off:** → Code Agent

**Comando asociado:** `/arch`

---

### 3.4 Code Agent

**Responsabilidad:** Implementar el código del feature por vertical slices, siguiendo estrictamente arch.md y los patrones del proyecto.

**Cuándo se activa:** Cuando arch.md y tasks.md están aprobados. Se ejecuta una vez por cada vertical slice.

**Inputs requeridos:**

| Input | Fuente | Obligatorio |
|-------|--------|:-----------:|
| arch.md | docs/features/FEAT-XXX/ | ✅ |
| tasks.md | docs/features/FEAT-XXX/ | ✅ |
| ui_contract.md | docs/features/FEAT-XXX/ | ✅ |
| Codebase existente | Repositorio | ✅ |

**Vertical Slices (orden obligatorio):**

**Slice 1 — Happy path + loading:**
- Implementar la UI con estado Loading y Content
- Conectar al API/datasource
- Generar tests unitarios para ViewModel y UseCase
- Commit con conventional commit: `feat(xxx): implement happy path`

**Slice 2 — Empty + error + retry:**
- Implementar estados Empty y Error con acción de retry
- Manejo de errores por código HTTP (401→refresh, 429→backoff, etc.)
- Tests de error handling
- Commit: `feat(xxx): add error handling and empty state`

**Slice 3 — Analytics + feature flag + a11y + i18n:**
- Instrumentar eventos de analytics
- Wrappear con feature flag
- Content descriptions / accessibility labels
- Strings en resources (no hardcoded)
- Commit: `feat(xxx): add analytics, a11y, and i18n`

**Reglas estrictas para el Code Agent:**

### DO:
- Seguir arch.md — no inventar arquitectura propia
- Usar componentes existentes del Design System
- Generar tests junto con la implementación
- Commits atómicos con conventional commits
- Ejecutar lint antes de cada commit
- Strings en resources/strings.xml o Localizable.strings

### DON'T:
- Crear nuevas dependencias sin que estén aprobadas en arch.md
- Ignorar los gates definidos en este AGENTS.md
- Generar código "demo" sin error handling real
- Hardcodear strings, colores o dimensiones
- Dejar TODOs sin issue asociado
- Usar `println` / `print` / `Log.d` para debugging (usar logger del proyecto)

**Validación antes de cada commit:**

```bash
# Android
./gradlew ktlintCheck
./gradlew testDebugUnitTest

# iOS
swiftlint
xcodebuild test -scheme App -destination 'platform=iOS Simulator,name=iPhone 16'
```

**Gates de validación (por slice):**

- [ ] Código compila sin warnings
- [ ] Lint pasa sin errores
- [ ] Tests unitarios pasan (cobertura mínima del ViewModel: 80%)
- [ ] No hay strings hardcodeados
- [ ] Commits siguen conventional commits
- [ ] `/security` ejecutado y sin FAIL críticos

**Hand-off:** → PR Guardian (después de completar todos los slices)

**Comandos asociados:** `/api`, `/ui`, `/security`

---

### 3.5 PR Guardian

**Responsabilidad:** Validar que el feature cumple con todos los estándares de calidad, seguridad y completitud antes de crear el PR.

**Cuándo se activa:** Cuando todos los vertical slices están implementados y sus gates individuales aprobados.

**Inputs requeridos:**

| Input | Fuente | Obligatorio |
|-------|--------|:-----------:|
| Código implementado | Branch del feature | ✅ |
| spec.md | docs/features/FEAT-XXX/ | ✅ |
| arch.md | docs/features/FEAT-XXX/ | ✅ |
| qa_plan.md | docs/features/FEAT-XXX/ | ✅ |

**Outputs generados:**

- **pr.md** — Documento del PR que incluye:
  - Descripción del cambio (qué y por qué)
  - Link al ticket (FEAT-XXX)
  - Screenshots/recordings de los 4 estados de UI
  - Lista de archivos modificados/creados con propósito
  - Breaking changes (si aplica)
  - Migration notes (si aplica)
  - Checklist de Definition of Done

**Definition of Done (DoD) — checklist completo:**

### Funcionalidad
- [ ] Todos los Acceptance Criteria de spec.md cumplidos
- [ ] 4 estados de UI implementados (Loading/Content/Error/Empty)
- [ ] Manejo de errores robusto (no crashes, no estados indefinidos)

### Calidad de código
- [ ] Lint pasa sin errores (ktlint / swiftlint)
- [ ] Tests unitarios pasan (cobertura ≥ 80% en ViewModels y UseCases)
- [ ] No hay TODOs sin issue
- [ ] Conventional commits en todos los commits

### Arquitectura
- [ ] `/arch` ejecutado y validado — Clean Architecture respetada
- [ ] No dependencias nuevas sin aprobación
- [ ] DTOs no expuestos fuera de data layer

### Seguridad (OWASP MASVS)
- [ ] `/security` ejecutado y sin FAIL críticos
- [ ] Datos sensibles en Keystore/Keychain (no SharedPreferences/UserDefaults)
- [ ] No PII en logs
- [ ] Network security config correcta (TLS 1.3, no cleartext)
- [ ] Deep links validados (si aplica)

### Accesibilidad
- [ ] Content descriptions / accessibility labels presentes
- [ ] Dynamic Type support
- [ ] Touch targets ≥ 48dp / 44pt
- [ ] Color contrast ≥ 4.5:1

### Observabilidad
- [ ] Analytics events implementados según spec.md
- [ ] Feature flag wrapping (si aplica)
- [ ] Crash reporting configurado

### Internacionalización
- [ ] Todos los strings en resources (no hardcoded)
- [ ] Plurales manejados correctamente
- [ ] RTL support (si aplica)

**Hand-off:** → Code Review humano → Merge

**Comando asociado:** `/pr`

---

## 4. Flujo Ligero (Lightweight)

Para bugs, hotfixes y cambios menores (< 1 día de trabajo):

**Solo requiere:**
- spec.md simplificado (descripción + AC + fix propuesto)
- PR checklist (DoD reducido)

**Salta:**
- ui_contract.md
- arch.md
- Vertical slices formales

**Criterio para usar flujo ligero:**
- Bugfix de < 50 líneas de código
- Cambio cosmético (copy, colores, spacing)
- Hotfix de producción urgente

---

## 5. Feedback Loop

### Retrospectiva por feature

**Trigger:** 1 semana post-release del feature a producción.

**Output:** `docs/features/FEAT-XXX/retro.md`

**Contenido:**
- ¿El spec fue preciso? ¿Hubo ambigüedades no detectadas?
- ¿La arquitectura propuesta se mantuvo o hubo desvíos?
- ¿El agente generó código que requirió refactoring significativo?
- ¿El /security detectó issues reales?
- ¿Qué mejoraría del flujo para el próximo feature?
- Métricas: tiempo total, número de iteraciones con el agente, bugs post-merge

### Actualización de decisiones

Si la retrospectiva revela un patrón (ej: "siempre tenemos que cambiar la forma en que manejamos errores de red"), se actualiza:
- `docs/decisions.md` con un nuevo ADR
- `rules/` con la nueva convención
- Skills relevantes con el aprendizaje

---

## 6. Enforcement

### En CI/CD
- PR falla si no existe `docs/features/FEAT-XXX/spec.md`
- PR falla si lint no pasa
- PR falla si tests no pasan
- PR falla si cobertura cae por debajo del umbral

### En code review
- Reviewer verifica que el PR tiene el checklist de DoD completo
- Reviewer verifica que `/security` fue ejecutado

### En onboarding
- Todo nuevo developer lee AGENTS.md como parte del onboarding
- Se asigna un feature pequeño para practicar el flujo completo

---

## 7. Herramientas Compatibles

| Propósito | Herramientas soportadas |
|-----------|------------------------|
| Tickets | Jira, Linear, GitHub Issues |
| Diseño | Figma (MCP + screenshots), Sketch, Adobe XD |
| Coding Agent | Claude Code, Cursor, GitHub Copilot, Windsurf |
| IDE | Android Studio, Xcode, VS Code |
| CI/CD | GitHub Actions, Bitrise, CircleCI, Fastlane |
| Feature Flags | LaunchDarkly, Firebase Remote Config |
| Analytics | Amplitude, Mixpanel, Firebase Analytics |
| Crash Reporting | Crashlytics, Sentry |

---

## 8. Checklist de Bootstrap (1 vez por proyecto)

Antes de usar este workflow:

- [ ] Este archivo (`AGENTS.md`) existe en la raíz del repo
- [ ] `CLAUDE.md` configurado con stack, convenciones y skills
- [ ] Directorio `.claude/skills/` con los 4 skills mínimos
- [ ] Directorio `.claude/commands/` con los 6 commands
- [ ] Directorio `rules/` con estándares del proyecto
- [ ] Directorio `docs/features/TEMPLATE/` con plantillas
- [ ] CI configurado con quality gates (lint + tests + cobertura)
- [ ] Design System documentado y tokens accesibles
- [ ] MCP de Figma configurado (opcional pero recomendado)

---

## 9. Changelog

| Fecha | Versión | Cambios |
|-------|---------|---------|
| 2026-02-15 | 2.0.0 | Integración de skills OWASP, API consume, arch-check, figma-to-ui. Unificación commands/skills. Flujo ligero. Feedback loop. |
| 2026-01-30 | 1.0.0 | Versión inicial con 5 agentes |

---

> **Nota:** Este documento es la fuente de verdad del sistema de desarrollo.
> Si hay conflicto entre este archivo y cualquier otro documento, AGENTS.md prevalece.
> Los skills y commands son ejecutores de los roles aquí definidos.

---
> Source: [LIDR-academy/mobile-facephi](https://github.com/LIDR-academy/mobile-facephi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
