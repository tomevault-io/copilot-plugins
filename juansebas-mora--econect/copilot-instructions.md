## econect

> EConect es una aplicación Android nativa que conecta ciudadanos comunes con recicladores profesionales.

# CLAUDE.md — EConect

## Descripción general del proyecto

EConect es una aplicación Android nativa que conecta ciudadanos comunes con recicladores profesionales.
Los ciudadanos registran material reciclable y su ubicación; los recicladores consultan rutas asignadas,
recogen el material y confirman la entrega con precio en el centro de acopio.

---

## Stack tecnológico

| Componente | Tecnología |
|---|---|
| Lenguaje | Kotlin |
| UI | Jetpack Compose + Material 3 |
| Arquitectura | MVVM + Clean Architecture (capas: UI → ViewModel → Repository → DataSource) |
| Navegación | Navigation Compose con deep links |
| Inyección de dependencias | Hilt |
| Base de datos local | Room |
| Preferencias | DataStore (Proto) |
| Backend / Auth | Firebase Auth (email + contraseña) |
| Base de datos remota | Cloud Firestore |
| Notificaciones push | Firebase Cloud Messaging (FCM) |
| Mapas y geolocalización | Google Maps SDK + Google Places API + FusedLocationProviderClient |
| Chat en tiempo real | Firestore (colección de mensajes con listeners en tiempo real) |
| Tareas en segundo plano | WorkManager |
| Imágenes | Coil |
| Testing | JUnit 4/5 · MockK · Turbine · Compose UI Test |
| Build | Gradle (Kotlin DSL) · compileSdk 35 · minSdk 24 |

---

## Arquitectura MVVM + Clean Architecture

```
app/
├── di/                          # Módulos Hilt (AppModule, FirebaseModule, etc.)
├── data/
│   ├── local/
│   │   ├── db/                  # Room Database, DAOs, Entities
│   │   └── datastore/           # DataStore Proto para preferencias de usuario
│   ├── remote/
│   │   ├── firebase/            # FirebaseAuth, Firestore data sources
│   │   └── fcm/                 # FCM service, token management
│   ├── repository/              # Implementaciones de repositorios
│   └── model/                   # DTOs y mapeadores de datos
├── domain/
│   ├── model/                   # Modelos de dominio (User, Material, Ruta, etc.)
│   ├── repository/              # Interfaces de repositorio
│   └── usecase/                 # Casos de uso (un archivo por caso de uso)
├── presentation/
│   ├── auth/                    # Login, Registro, recuperación de contraseña
│   ├── citizen/
│   │   ├── dashboard/           # Dashboard ciudadano
│   │   ├── material/            # Añadir / ver material reciclable
│   │   └── profile/             # Perfil ciudadano
│   ├── recycler/
│   │   ├── dashboard/           # Dashboard reciclador
│   │   ├── routes/              # Rutas asignadas y mapa
│   │   ├── pickup/              # Recoger material, confirmar cantidad y precio
│   │   └── profile/             # Perfil reciclador
│   ├── shared/
│   │   ├── chat/                # Chat entre reciclador y ciudadano
│   │   ├── notifications/       # Centro de notificaciones
│   │   ├── history/             # Historial de transacciones
│   │   └── rating/              # Sistema de calificaciones
│   └── navigation/              # NavGraph, rutas, argumentos
└── util/                        # Extensions, constantes, formatters
```

### Reglas por capa

- **UI (Composables):** Solo observa StateFlow/LiveData del ViewModel. No tiene lógica de negocio ni acceso directo a repositorios.
- **ViewModel:** Expone `UiState` como `StateFlow<T>`. Llama únicamente a casos de uso o repositorios. No importa clases de `data/`.
- **Domain (UseCases):** Clases con un único método `invoke()`. Sin dependencias de Android framework.
- **Repository:** La interfaz vive en `domain/`; la implementación en `data/`. Decide entre caché local y fuente remota.
- **DataSource:** Acceso directo a Room, Firestore o FCM. Sin lógica de negocio.

---

## Modelos de dominio clave

```kotlin
// Tipos de usuario
enum class UserType { CITIZEN, RECYCLER }

data class User(
    val id: String,
    val name: String,
    val email: String,
    val phone: String,
    val userType: UserType,
    val preferredLocations: List<LatLng>,
    val availableSchedules: List<Schedule>,
    val rating: Float,
    val ratingCount: Int
)

data class RecyclableMaterial(
    val id: String,
    val citizenId: String,
    val type: MaterialType,         // PAPER, PLASTIC, GLASS, METAL, etc.
    val condition: MaterialCondition,
    val quantity: MaterialQuantity, // kg o unidades (cajas, botellas, etc.)
    val pickupLocation: LatLng,
    val status: MaterialStatus,     // AVAILABLE, ASSIGNED, COLLECTED
    val createdAt: Timestamp
)

data class Route(
    val id: String,
    val recyclerId: String,
    val stops: List<RouteStop>,     // Cada stop tiene material + ciudadano + horario
    val date: LocalDate,
    val status: RouteStatus         // PENDING, IN_PROGRESS, COMPLETED
)

data class Transaction(
    val id: String,
    val routeId: String,
    val recyclerId: String,
    val citizenId: String,
    val materialId: String,
    val confirmedQuantity: MaterialQuantity,
    val pricePerUnit: Double,
    val totalAmount: Double,
    val recyclingCenterId: String,
    val completedAt: Timestamp
)

data class ChatMessage(
    val id: String,
    val routeId: String,
    val senderId: String,
    val content: String,
    val timestamp: Timestamp,
    val read: Boolean
)

data class Rating(
    val id: String,
    val fromUserId: String,
    val toUserId: String,
    val routeId: String,
    val score: Int,    // 1-5
    val comment: String,
    val createdAt: Timestamp
)
```

---

## Convenciones de código

- **Idioma del código:** inglés (nombres de clases, variables, funciones).
- **Idioma de comentarios y strings de UI:** español.
- **Nombrado:**
    - Clases: `PascalCase`
    - Funciones y variables: `camelCase`
    - Constantes: `UPPER_SNAKE_CASE`
    - Composables: `PascalCase`, prefijo descriptivo (ej: `CitizenDashboardScreen`)
- **UiState pattern:** cada ViewModel expone `data class XUiState(val isLoading, val error, val data)`.
- **Coroutines:** `viewModelScope` en ViewModels; `Dispatchers.IO` en repositorios.
- **Manejo de errores:** clase sellada `Result<T>` propia en el dominio; nunca exponer excepciones a la UI directamente.
- **No usar** `!!` (non-null assertion). Usar `?.let`, `?: return`, o `requireNotNull` con mensaje descriptivo.
- **Formateo:** ktlint con configuración por defecto.

---

## Seguridad

- Autenticación exclusivamente vía Firebase Auth (email + contraseña). No almacenar contraseñas localmente.
- Las reglas de Firestore Security Rules deben validar `request.auth.uid` antes de permitir cualquier lectura o escritura.
- Los tokens FCM se rotan al hacer logout.
- No loggear PII (emails, teléfonos, ubicaciones) en Logcat en builds de release.
- Usar `BuildConfig.DEBUG` para guards de logs.
- Permisos de ubicación: solicitar solo `ACCESS_FINE_LOCATION` cuando el mapa esté visible; nunca en background permanente.

---

## Firebase — estructura de Firestore

```
users/{userId}
  ├── type: "citizen" | "recycler"
  ├── name, email, phone, rating, ratingCount
  ├── preferredLocations: [{lat, lng, label}]
  └── schedules: [{dayOfWeek, startTime, endTime}]

materials/{materialId}
  ├── citizenId, type, condition, quantity, unit
  ├── pickupLocation: {lat, lng}
  └── status: "available" | "assigned" | "collected"

routes/{routeId}
  ├── recyclerId, date, status
  └── stops: [{materialId, citizenId, scheduledTime, location}]

transactions/{transactionId}
  └── routeId, recyclerId, citizenId, materialId,
      confirmedQuantity, pricePerUnit, totalAmount, completedAt

chats/{routeId}/messages/{messageId}
  └── senderId, content, timestamp, read

ratings/{ratingId}
  └── fromUserId, toUserId, routeId, score, comment, createdAt
```

---

## Consideraciones de UX / Accesibilidad

- La app está orientada a usuarios con **bajo nivel de alfabetización digital**. Priorizar:
    - Iconografía clara y etiquetas de texto en todos los botones.
    - Texto mínimo de 16sp en cuerpo, 14sp en secundario.
    - Contrastes AA (WCAG 2.1) en todos los colores de texto.
    - Flujos con el menor número posible de pasos (máximo 3 para acciones frecuentes).
    - Mensajes de error en lenguaje simple, sin tecnicismos.
- Soporte para modo oscuro desde el inicio.
- Soporte para texto grande del sistema (`fontScale`).

---

## Testing

- **Unit tests:** todos los UseCases y ViewModels deben tener cobertura mínima del 80%.
- **Integration tests:** repositorios contra Room (in-memory) y Firestore emulador.
- **UI tests:** flujos críticos con Compose UI Test: login, registro de material, confirmación de recogida.
- Usar **Turbine** para testear `StateFlow` en ViewModels.
- Usar **MockK** para mocks; preferir fakes sobre mocks para repositorios en tests de ViewModel.

---

## Comandos frecuentes

```bash
# Build debug
./gradlew assembleDebug

# Run unit tests
./gradlew test

# Run Android tests (requiere emulador)
./gradlew connectedAndroidTest

# Lint
./gradlew lint

# ktlint check
./gradlew ktlintCheck

# ktlint format
./gradlew ktlintFormat
```

---

## Variables de entorno / configuración

- `google-services.json`: archivo de configuración Firebase. No commitear. Obtener de Firebase Console.
- `local.properties`: API keys (Google Maps). No commitear.
- Usar `BuildConfig` fields para distinguir entornos (`DEV`, `PROD`).
- Firebase Emulator Suite disponible para desarrollo local: Auth (9099), Firestore (8080), FCM (mock manual).

---

## Notas para Claude Code

- Antes de crear cualquier archivo, revisar si ya existe una clase similar en el módulo correspondiente.
- Al agregar una dependencia nueva en `build.gradle.kts`, verificar compatibilidad con el `compileSdk` y `minSdk` actuales.
- Cada nueva pantalla requiere: Composable Screen + ViewModel + UiState + ruta en NavGraph.
- Los casos de uso deben ser clases con `operator fun invoke()`, no funciones sueltas.
- Al modificar el esquema de Firestore, actualizar también los mapeadores en `data/model/`.
- Preferir `LazyColumn` sobre `Column` con scroll para listas de más de 5 elementos.
- No usar `rememberCoroutineScope` para operaciones que deben sobrevivir a recomposiciones; usar el ViewModel.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juansebas-mora) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-15 -->
