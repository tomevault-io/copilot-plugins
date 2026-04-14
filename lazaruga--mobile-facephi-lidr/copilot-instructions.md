## mobile-facephi-lidr

> **AGENTS.md es la fuente de verdad del sistema.** Si hay conflicto entre este archivo y AGENTS.md, prevalece AGENTS.md. Leer AGENTS.md antes de iniciar cualquier feature nueva.

# CLAUDE.md — Proyecto Mobile Nativo

## Precedencia

**AGENTS.md es la fuente de verdad del sistema.** Si hay conflicto entre este archivo y AGENTS.md, prevalece AGENTS.md. Leer AGENTS.md antes de iniciar cualquier feature nueva.

## Idioma

Todas las respuestas, comentarios de código, mensajes de commit, descripciones de PR y documentación generada deben estar en **español**.

## Proyecto
App móvil nativa — Android (Kotlin + Jetpack Compose) / iOS (Swift + SwiftUI).
Arquitectura: Clean Architecture + MVI (Android) / MVVM (iOS).
Monorepo con módulos por feature.

## Stack

### Android
- Kotlin 2.x, Jetpack Compose (Material 3), Hilt (DI)
- Retrofit + OkHttp (networking), Room (local DB)
- Coroutines + Flow (async), Coil (imágenes)
- Gradle con version catalogs (libs.versions.toml)

### iOS
- Swift 5.9+, SwiftUI, Combine
- URLSession + async/await (networking), SwiftData o CoreData (local DB)
- Swift Package Manager (dependencias)
- Keychain wrapper para almacenamiento seguro

### Compartido
- CI/CD: GitHub Actions + Fastlane
- Feature Flags: Firebase Remote Config
- Analytics: Firebase Analytics / Amplitude
- Crash Reporting: Crashlytics / Sentry

## Comandos frecuentes

```bash
# Android
./gradlew assembleDebug                    # Build debug
./gradlew testDebugUnitTest                # Tests unitarios
./gradlew ktlintCheck                      # Lint
./gradlew ktlintFormat                     # Auto-format
./gradlew connectedAndroidTest             # Tests de integración

# iOS
xcodebuild build -scheme App -destination 'platform=iOS Simulator,name=iPhone 16'
xcodebuild test -scheme App -destination 'platform=iOS Simulator,name=iPhone 16'
swiftlint                                  # Lint
swiftlint --fix                            # Auto-format

# General
git log --oneline -10                      # Últimos commits
git diff --name-only                       # Archivos modificados
```

## Arquitectura

Cada feature sigue Clean Architecture con esta estructura:

```
feature-xxx/
  presentation/     → Screens, ViewModels, UiStates (solo depende de domain/)
  domain/           → Models, UseCases, Repository interfaces (NO importa de nadie)
  data/             → Repository impl, DTOs, API services, DAOs (implementa domain/)
```

Regla de dependencia (NUNCA violar):
- `presentation → domain ← data`
- domain/ NUNCA importa de presentation/ ni data/
- presentation/ NUNCA importa directamente de data/
- DTOs NUNCA se exponen fuera de data/

### Patrones obligatorios
- ViewModels exponen un solo UiState (sealed class / enum)
- Toda pantalla tiene 4 estados: Loading, Content, Error, Empty
- UseCases tienen una sola responsabilidad (operator invoke)
- Repositories se inyectan como interface, nunca implementación concreta
- Mappers explícitos entre capas: DTO ↔ Domain Model ↔ Entity

## Convenciones de código

### Naming
- Archivos: PascalCase (UserProfileScreen.kt, UserProfileView.swift)
- Funciones/variables: camelCase
- Constantes: UPPER_SNAKE_CASE
- Packages/modules: lowercase.separated.by.dots

### Commits
Conventional Commits obligatorio:
- `feat(user-profile): add avatar upload`
- `fix(auth): handle token expiration on background`
- `refactor(networking): extract retry logic to interceptor`
- `test(payments): add unit tests for PaymentUseCase`

### Strings
- NUNCA hardcodear strings visibles al usuario
- Android: strings.xml con naming `screen_element_description` (ej: `profile_button_save`)
- iOS: Localizable.strings o String Catalogs con la misma convención
- Plurales manejados con quantity strings (Android) / stringsdict (iOS)

### Imports
- No wildcard imports (no `import java.util.*`)
- Ordenar: stdlib → third-party → proyecto

## Seguridad (OWASP MASVS)

IMPORTANTE: Consultar `.claude/skills/owasp-mobile/SKILL.md` para revisiones completas.

### Reglas que SIEMPRE aplican:
- Datos sensibles (tokens, passwords, PII) → Android Keystore / iOS Keychain
- NUNCA en SharedPreferences, UserDefaults, ni logs
- Network: TLS 1.3 obligatorio, certificate pinning habilitado
- No permitir cleartext traffic (network_security_config.xml / ATS)
- No loggear datos PII en ningún entorno (ni debug)
- Deep links: validar con Universal Links (iOS) / App Links (Android)
- WebViews: JavaScript deshabilitado por defecto, habilitar solo si necesario
- Permisos: solicitar el mínimo necesario, en el momento de uso
- ProGuard/R8 habilitado en release (Android)
- No exponer Activities/ContentProviders sin android:exported=false

### Credenciales y secrets
- NUNCA hardcodear API keys en el código fuente
- Usar BuildConfig (Android) / xcconfig (iOS) para inyectar en build time
- Secrets de CI en GitHub Secrets, nunca en el repo

## Testing

### Cobertura mínima
- ViewModels: 80%
- UseCases: 90%
- Mappers: 100%
- UI: Snapshot tests para los 4 estados (Loading/Content/Error/Empty)

### Frameworks
- Android: JUnit 5, Mockk, Turbine (para Flows), Compose UI Testing
- iOS: XCTest, Swift Testing, ViewInspector (para SwiftUI)

### Convención de naming para tests
```
fun `should return user profile when API returns 200`()
func test_shouldReturnUserProfile_whenAPIReturns200()
```

## Gotchas del proyecto

- NUNCA modificar archivos .pbxproj con Claude Code. Crear archivos y agregarlos manualmente en Xcode
- El módulo `legacy-auth` usa XML Views (no Compose). No migrar sin ticket explícito
- El endpoint /v1/users requiere header `X-App-Version` — sin él devuelve 403
- Room migrations son manuales (no auto-migration). Siempre crear migration file
- SwiftData solo disponible a partir de iOS 17 — verificar min deployment target

## Skills disponibles

Invocar con `/nombre` o Claude los detecta automáticamente por contexto:

| Skill | Descripción | Cuándo se activa |
|-------|-------------|-----------------|
| `/owasp-review` | Revisión de seguridad OWASP MASVS | Código que maneja datos sensibles, auth, networking |
| `/api-consume` | Consumo seguro de endpoints REST | Nuevo endpoint o modificación de networking |
| `/arch-check` | Validación de Clean Architecture | Review de PRs o generación de features |
| `/figma-to-ui` | Traducción de diseño a código nativo | Construcción de UI desde Figma/screenshots |

## Commands disponibles

| Command | Uso |
|---------|-----|
| `/spec FEAT-XXX` | Genera spec atómica desde ticket |
| `/ui` | Traduce diseño pegado a Composable/SwiftUI |
| `/api METHOD /path {body}` | Genera DTO, mapper, repo, tests |
| `/security` | Auditoría OWASP sobre archivos modificados |
| `/arch` | Valida dependencias y estructura |
| `/pr` | Prepara PR con DoD checklist |

## Workflows

El flujo completo de desarrollo está definido en **AGENTS.md** con 5 etapas y gates explícitos. Resumen rápido:

### Para una feature nueva (flujo completo — AGENTS.md §2-3)
1. **Spec Agent** (`/spec FEAT-XXX`): Ticket → spec.md + risks.md + tasks.md + qa_plan.md
   - Gate: ACs testables (Given/When/Then) + 4 estados UI definidos
2. **Design Translator** (`/ui`): Figma/screenshot + spec.md → ui_contract.md
   - Gate: State machine completa + tokens Design System + a11y checklist
3. **Mobile Architect** (`/arch`): spec + ui_contract → arch.md + tasks.md refinado
   - Gate: Clean Architecture validada + dependencias justificadas
4. **Code Agent**: Implementar por vertical slices (en orden):
   - Slice 1: Happy path + loading → `feat(xxx): implement happy path`
   - Slice 2: Empty + error + retry → `feat(xxx): add error handling and empty state`
   - Slice 3: Analytics + feature flag + a11y + i18n → `feat(xxx): add analytics, a11y, and i18n`
   - Gate por slice: lint pass + tests pass + `/security` sin FAIL críticos
5. **PR Guardian** (`/pr`): Validar DoD completo → PR listo para merge
   - Gate: Funcionalidad + calidad + arquitectura + seguridad + a11y + i18n

### Para un bugfix (flujo ligero — AGENTS.md §4)
Criterio: < 50 líneas, cambio cosmético o hotfix urgente.
1. Reproducir el bug (describir pasos)
2. Identificar root cause con tests que fallan
3. Implementar fix
4. Agregar test de regresión
5. Ejecutar `/security` sobre archivos modificados
6. Commit con `fix(modulo): descripción`

### Para code review
1. Ejecutar `/arch` sobre los archivos del PR
2. Ejecutar `/security` sobre los archivos del PR
3. Verificar DoD checklist completo (AGENTS.md §3.5)
4. Verificar conventional commits
5. Verificar que no hay strings hardcodeados

### Artefactos por feature
Todos los documentos se generan en `docs/features/FEAT-XXX/`:
- spec.md, ui_contract.md, arch.md, tasks.md, qa_plan.md, risks.md, pr.md
- Post-release: retro.md (1 semana después — AGENTS.md §5)

## Referencias

- **AGENTS.md** — Source of truth: roles, contratos, inputs/outputs, hand-offs y gates
- **rules/** — Estándares detallados por plataforma (engineering, android, ios, security)
- **docs/features/** — Documentación por feature (spec, arch, tasks, qa, pr)
- **docs/decisions.md** — ADRs (Architecture Decision Records)
- **.claude/skills/** — Skills: owasp-mobile, api-consume, arch-check, figma-to-ui
- **.claude/commands/** — Commands: spec, api, security, ui, arch, pr

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lazaruga)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lazaruga)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
