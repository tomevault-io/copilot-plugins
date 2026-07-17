## api-movements

> Backend de gestión de finanzas personales. Permite registrar movimientos, suscripciones, ingresos y cuentas compartidas con actualizaciones en tiempo real vía WebSocket.

# api-movements — AGENTS.md

Backend de gestión de finanzas personales. Permite registrar movimientos, suscripciones, ingresos y cuentas compartidas con actualizaciones en tiempo real vía WebSocket.

> **Producción:** `https://movement.eva-core.com`

---

## Tech Stack

| Área | Tecnología |
|---|---|
| Language | Java 25 |
| Framework | Spring Boot 4.0.2 / Gradle 9 |
| Database | MySQL 8 + Liquibase (`ddl-auto: none`) |
| ORM | Spring Data JPA / Hibernate |
| Auth | Keycloak — OAuth2 Resource Server, JWT RS256 |
| Messaging | RabbitMQ (AMQP) + WebSocket STOMP/SockJS |
| Mapping | MapStruct 1.6.3 (`componentModel = "spring"`) |
| Boilerplate | Lombok (`@Data`, `@Builder`, `@RequiredArgsConstructor`) |
| Cache | Caffeine (in-memory, 5h TTL para currency) |
| PDF parsing | Apache PDFBox 3.0.6 |
| API docs | SpringDoc OpenAPI 3 (`/docs`) |
| Testing | Spock 2.4 + Testcontainers (MySQL) + Mockito |
| CI/CD | GitHub Actions → Docker Hub `mferrerovilas/api-movements` (ARM64) |

---

## Regla de Oro: Tests y Checkstyle Obligatorios

> **Todo cambio en el código de producción debe ir acompañado de la ejecución de tests y checkstyle antes de considerarse completo.**

```bash
# Ejecutar tests
./gradlew test

# Ejecutar checkstyle
./gradlew checkstyleMain checkstyleTest

# O ambos juntos
./gradlew test checkstyleMain checkstyleTest
```

- **No se acepta código nuevo sin cobertura** de los casos principales.
- **No se acepta código que rompa tests existentes** sin justificación explícita.
- **No se acepta código que viole el checkstyle.** Si la regla es incorrecta, se discute antes de saltearla.

---

## Package Structure

```
api.m2.movements
├── controller/         REST controllers — uno por dominio, todos bajo /v1/*
├── entities/           JPA entities (Lombok @Data + @Builder)
├── enums/              Enums de dominio: WorkspaceRole, CategoryEnum, EventType, etc.
├── exceptions/         BusinessException, EntityNotFoundException, PermissionDeniedException
├── helpers/            PDF parsers: PdfExtractprHelper (interface) + BBVA/Galicia impls + ParserRegistry
├── mappers/            MapStruct interfaces, 13 mappers, se componen entre sí
├── projections/        JPA interface projections (read-only, para queries livianas)
├── records/            DTOs como Java records, organizados por dominio
│   ├── workspaces/
│   ├── balance/
│   ├── invite/
│   ├── movements/
│   ├── users/          UserBaseRecord, UserMeRecord
│   └── ...
├── repositories/       Spring Data JPA — todos extienden JpaRepository
├── security/           JwtAuthenticationConverter + SecurityConfiguration
└── services/           Lógica de negocio, organizada por dominio
    ├── balance/
    ├── workspaces/     WorkspaceQueryService (reads) + WorkspaceAddService (writes) + MembershipService
    ├── movements/      MovementAddService + MovementGetService + MovementFactory + file import strategies
    ├── publishing/
    │   ├── rabbit/     RabbitSocketMessageService (base) + MovementPublishServiceRabbit
    │   └── websockets/ WebSocketMessageService (base) + Movement/Workspace/ServicePublishServiceWebSocket
    └── user/           UserService + UserAddService
```

---

## Entidades y Relaciones Clave

| Entity | Campos relevantes | Relaciones |
|---|---|---|
| `User` | `id: Long` (PK auto-increment), `email`, `isFirstLogin`, `userType` | base de todo |
| `Workspace` | `id`, `name` | `owner → User`, `members → WorkspaceMember[]` |
| `WorkspaceMember` | `role: WorkspaceRole` | `user → User`, `workspace → Workspace` |
| `WorkspaceInvitation` | `status: InvitationStatus` | `user → User` (invitado), `invitedBy → User`, `workspace → Workspace` |
| `Movement` | `amount`, `date`, `type`, `description`, `cuotaActual/Total` | `owner → User`, `workspace → Workspace`, `category`, `currency`, `bank` |
| `Income` | `amount` | `user → User`, `bank → Bank`, `currency`, `workspace → Workspace` |
| `Subscription` | `description`, `amount`, `lastPayment`, `@Transient isPaid()` | `owner → User`, `workspace → Workspace`, `currency` |
| `UserBank` | `createdAt` | `user → User`, `bank → Bank` |
| `UserSetting` | `settingKey: UserSettingKey`, `settingValue: Long` | `user → User` |

> **IMPORTANTE:** `User.id` es un `Long` auto-incremental de DB. El Keycloak subject (`sub` del JWT) es un UUID `String` separado. **No son intercambiables.**

---

## API Endpoints

### Usuarios y Onboarding
| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/v1/users/me` | Retorna `UserMeRecord`. Si es nuevo: `{ id: null, isFirstLogin: true }` |
| `POST` | `/v1/onboarding` | Crea usuario, cuentas e ingreso inicial |

### Movimientos
| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/v1/expenses` | Movimientos paginados con filtros |
| `POST` | `/v1/expenses` | Crear movimiento |
| `POST` | `/v1/expenses/import-file` | Importar movimientos desde PDF bancario |
| `PATCH` | `/v1/expenses/{id}` | Actualización parcial (MapStruct `IGNORE` null) |
| `DELETE` | `/v1/expenses/{id}` | Eliminar movimiento |

### Balance
| Método | Endpoint | Descripción |
|---|---|---|
| `GET` | `/v1/balance` | Balance total (INGRESO/GASTO) |
| `GET` | `/v1/balance/category` | Balance por categoría |
| `GET` | `/v1/balance/group` | Balance por cuenta |
| `GET` | `/v1/balance/monthly-evolution` | Evolución mensual por moneda |

### Workspaces
| Método | Endpoint | Descripción |
|---|---|---|
| `POST` | `/v1/workspace` | Crear workspace |
| `GET` | `/v1/workspace/membership` | Membresías del usuario |
| `GET` | `/v1/workspace/count` | Workspaces con cantidad de miembros |
| `DELETE` | `/v1/workspace/{workspaceId}` | Salir de un workspace |
| `POST` | `/v1/workspace/{id}/invitations` | Invitar usuarios por email |
| `GET` | `/v1/workspace/invitations` | Invitaciones pendientes del usuario |
| `PATCH` | `/v1/workspace/invitations/{invitationId}` | Aceptar/rechazar invitación |
| `PATCH` | `/v1/workspace/{id}/default` | Setear workspace por defecto |

### Ingresos, Suscripciones, Settings y Bancos
| Método | Endpoint | Descripción |
|---|---|---|
| `GET/POST` | `/v1/income` | Listar / crear ingreso |
| `DELETE` | `/v1/income/{id}` | Eliminar ingreso |
| `POST` | `/v1/income/{id}/reload` | Recargar ingreso |
| `GET/POST` | `/v1/subscriptions` | Listar / crear suscripción |
| `PATCH` | `/v1/subscriptions/{id}/payment` | Registrar pago |
| `PATCH/DELETE` | `/v1/subscriptions/{id}` | Actualizar / eliminar suscripción |
| `GET` | `/v1/settings/defaults` | Configuración por defecto del usuario |
| `GET/PUT` | `/v1/settings/{key}` | Leer / actualizar setting específico |
| `GET` | `/v1/banks` | Bancos asociados al usuario |
| `POST` | `/v1/banks` | Agregar banco al usuario (idempotente) |
| `DELETE` | `/v1/banks/{id}` | Quitar banco de la lista del usuario |

---

## WebSocket (STOMP)

**Endpoint SockJS:** `/ws`  
Todos los mensajes se envuelven en `EventWrapper<T> { eventType: EventType, message: T }`.

> **CRÍTICO — tipos de ID usados en los topics:**
> - **Movimientos / servicios / workspaces:** usan `workspaceId` (`Long`, PK de `Workspace`)
> - **Invitaciones:** usan `userId` (`Long`, PK de `User` en DB — **no** el Keycloak subject)
> - **Default workspace:** usa el Keycloak `sub` UUID (`String`) — único caso que usa el subject

| Topic | EventType | Payload | Cuándo |
|---|---|---|---|
| `/topic/movimientos/{workspaceId}/new` | `MOVEMENT_ADDED` | `MovementRecord` | Movimiento creado o importado |
| `/topic/movimientos/{workspaceId}/delete` | `MOVEMENT_DELETED` | `Long` (movementId) | Movimiento eliminado |
| `/topic/servicios/{workspaceId}/new` | `SERVICE_PAID` | `SubscriptionRecord` | Suscripción creada |
| `/topic/servicios/{workspaceId}/update` | `SERVICE_PAID / SERVICE_UPDATED` | `SubscriptionRecord` | Suscripción pagada o actualizada |
| `/topic/servicios/{workspaceId}/remove` | `SERVICE_DELETED` | `SubscriptionRecord` | Suscripción eliminada |
| `/topic/invitation/{userId}/new` | `INVITATION_ADDED` | `InvitationToWorkspaceRecord` | Invitación enviada |
| `/topic/invitation/{userId}/update` | `INVITATION_CONFIRMED_REJECTED` | `InvitationToWorkspaceRecord` | Invitación aceptada/rechazada |
| `/topic/workspace/{ownerId}/new` | `WORKSPACE_CREATED` | `WorkspaceRecord` | Workspace creado |
| `/topic/workspace/{workspaceId}/leave` | `WORKSPACE_LEFT` | `WorkspaceRecord` | Miembro salió del workspace |
| `/topic/workspace/{workspaceId}/members/update` | `MEMBERSHIP_UPDATED` | `WorkspaceDetail` | Miembro agregado al workspace |
| `/topic/workspace/default/{keycloakSubject}` | `MEMBERSHIP_UPDATED` | `WorkspaceDetail` | Workspace por defecto cambiado |

Todos los publishers usan `@TransactionalEventListener(phase = AFTER_COMMIT)` — el push WS solo ocurre si el commit fue exitoso.

---

## Seguridad

- **Principal:** el claim `preferred_username` del JWT (email) se mapea a `Authentication.getName()`. Con ese email se busca el `User` en DB.
- **Keycloak subject:** disponible vía `userService.getCurrentKeycloakId()`. Se usa **solo** para el topic `/topic/workspace/default/{id}`.
- **Roles:** se leen de `realm_access.roles[]` del JWT. Requiere prefijo `ROLE_`.

| Rutas | Acceso |
|---|---|
| `/swagger-ui/**`, `/v3/api-docs/**`, `/ws/**` | Públicas |
| `/v1/onboarding/**` | Requiere rol `ADMIN`, `FAMILY` o `GUEST` |
| Resto | Cualquier JWT válido |

**CORS permitido:** `https://movement.eva-core.com`, `http://localhost:5173`, `http://localhost:8081`

---

## Patrones de Diseño

### Service Splitting
Cada dominio tiene servicios separados por responsabilidad:
- `*AddService` — escritura
- `*GetService` / `*QueryService` — lectura
- `*Factory` — construcción de entidades

No existe ningún servicio monolítico tipo `MovementService`.

### Factory + Resolver
`MovementFactory` construye la entidad `Movement` resolviendo todas las FKs (category, currency, bank, user, workspace) a través de `CategoryResolver` y `CurrencyResolver`. `CurrencyResolver` usa cache Caffeine.

### Strategy (File Import)
`ExpenseFileStrategy` es clase abstracta. `BBVACreditImportService` y `GaliciaCreditImportService` se registran como beans. `MovementImportFileService` despacha por `match(bank)`.

### Event-Driven WebSocket
Los servicios de escritura publican Spring Application Events dentro de `@Transactional`. Los publishers WS escuchan con `@TransactionalEventListener(phase = AFTER_COMMIT)`.

### AOP — Membership Guard (`@RequiresMembership`)
Todo método de mutación sobre un recurso de cuenta compartida **debe** anotarse con `@RequiresMembership`. El aspecto `MembershipCheckAspect` verifica que el usuario sea miembro antes de ejecutar.

```java
// El id está en el índice 0 por defecto
@RequiresMembership(domain = MembershipDomain.INCOME)
public void deleteIncome(Long id) { ... }

// Cuando el id NO está en el primer parámetro, indicar el índice
@RequiresMembership(domain = MembershipDomain.MOVEMENT, idParamIndex = 1)
public void updateMovement(@Valid ExpenseToUpdate dto, Long id) { ... }
```

**Dominios disponibles:** `MOVEMENT`, `INCOME`, `SUBSCRIPTION`, `BUDGET`.  
Para agregar un nuevo dominio: extender `MembershipDomain` y crear un resolver que implemente `WorkspaceIdResolver`.

---

## Jobs Programados (Cron)

| Job | Cron | Descripción |
|-----|------|-------------|
| `MonthlySummaryJob` | `0 0 23 L * *` (último día del mes, 23:00) | Genera snapshots mensuales de resumen financiero para usuarios con `MONTHLY_SUMMARY_ENABLED` |
| `RecurringIncomeJob` | `0 0 6 1 * *` (día 1 de cada mes, 06:00) | Genera movimientos de ingreso automáticos para usuarios con `AUTO_INCOME_ENABLED` |

### UserSettingKey para Jobs

| Setting | Valor | Efecto |
|---------|-------|--------|
| `MONTHLY_SUMMARY_ENABLED` | `1` | Usuario recibe snapshot mensual automático |
| `AUTO_INCOME_ENABLED` | `1` | Los `Income` del usuario generan movimientos automáticamente el día 1 |

### Flujo de RecurringIncomeJob

1. Obtiene usuarios con `AUTO_INCOME_ENABLED = 1`
2. Para cada usuario, obtiene todos sus `Income` configurados
3. Por cada `Income`, genera un `Movement` de tipo `INGRESO` con:
   - Monto del `Income`
   - Fecha actual (UTC)
   - Descripción: "Ingreso recurrente"
   - Categoría: HOGAR
   - Workspace del `Income`

> **IMPORTANTE:** `Income` representa ingresos **recurrentes fijos** (salario, pensión). Los ingresos variables/puntuales deben cargarse directamente como `Movement` de tipo `INGRESO`.

---

## Convenciones del Proyecto

| Convención | Detalle |
|---|---|
| **DTOs como Java records** | Nunca clases para request/response. Organizados en `records/` por subpaquete. |
| **Dominio en español** | Enums, campos y comentarios en español: `INGRESO`, `GASTO`, `HOGAR`, etc. |
| **Liquibase es dueño del schema** | `ddl-auto: none`. Nunca modificar el schema sin un changeset nuevo. |
| **Sin prefix global en controllers** | Cada controller declara su propio `/v1/*` en `@RequestMapping`. |
| **Audit fields vía Hibernate** | `@CreationTimestamp` / `@UpdateTimestamp`, no Spring Data `@CreatedDate`. |
| **Métodos privados con `this.`** | Distingue llamadas propias de llamadas a dependencias inyectadas. |
| **`@EqualsAndHashCode` + `@ToString`** | Obligatorio en entidades bidireccionales para evitar recursión infinita. |
| **RabbitMQ** | Exchange `movement.topic`, routing key `n8n.import.file` → integración con n8n. |

---

## MapStruct

- Todos los mappers usan `componentModel = "spring"`.
- Se componen entre sí (ej: `MovementMapper` usa `CategoryMapper`, `CurrencyMapper`, `UserMapper`).
- Actualizaciones parciales: `@BeanMapping(nullValuePropertyMappingStrategy = IGNORE)` en métodos `update*()`.

### Mappers en tests — nunca mockear

```groovy
// Mapper sin dependencias
BankMapper bankMapper = Mappers.getMapper(BankMapper)

// Mapper con dependencias
MovementMapper movementMapper

def setup() {
    movementMapper = new MovementMapperImpl()
    ReflectionTestUtils.setField(movementMapper, "categoryMapper", Mappers.getMapper(CategoryMapper))
    ReflectionTestUtils.setField(movementMapper, "currencyMapper", Mappers.getMapper(CurrencyMapper))
    ReflectionTestUtils.setField(movementMapper, "userMapper", Mappers.getMapper(UserMapper))
}
```

MapStruct con `componentModel = "spring"` genera field injection (`@Autowired`), no constructor injection. `Mappers.getMapper()` solo funciona para mappers sin dependencias.

---

## Testing

### Reglas generales

- **TDD obligatorio** — todo componente nuevo (service, resolver, factory, helper) debe tener su test escrito junto con el código.
- **Sin `@SpringBootTest`** — tests unitarios puramente en memoria, sin contexto Spring.
- **Instanciación manual** — el servicio bajo test se crea vía constructor en `setup()`.
- **Estructura `given/when/then`** — obligatoria en todos los tests.
- **Nombres en inglés** — formato: `"metodo - should comportamiento"`.

### `Mock()` vs `Stub()`

| | `Mock()` | `Stub()` |
|---|---|---|
| Cuándo usar | Verificar que una interacción ocurrió (`1 * service.method(...)`) | Solo se necesita un valor de retorno |

### Wildcards tipados

Al usar `_` como matcher, siempre especificar el tipo. Nunca dejar `_` sin tipo cuando este es conocido.

```groovy
// ❌ MAL
1 * service.save(_)

// ✅ BIEN
1 * service.save(_ as MovementToAdd)
```

### Ubicación y naming

| Tipo | Ubicación |
|---|---|
| Servicios | `src/test/groovy/api/m2/movements/unit/services/` |
| Resolvers / utilitarios | `src/test/groovy/api/m2/movements/unit/unit/` |
| Nombre | `{NombreClase}Test.groovy` |

### Ejemplo canónico

```groovy
class SettingServiceTest extends Specification {

    MovementAddService movementAddService = Mock(MovementAddService)
    WorkspaceQueryService workspaceQueryService = Mock(WorkspaceQueryService)
    SettingService service

    def setup() {
        service = new SettingService(movementAddService, workspaceQueryService)
    }

    def "addIngreso - should save movement with correct parameters"() {
        given:
        def incomeToAdd = new IncomeToAdd("GALICIA", "EUR", new BigDecimal("1000.00"), "Mi workspace")
        workspaceQueryService.findWorkspaceByName("Mi workspace") >> Stub(Workspace) { getId() >> 1L }

        when:
        service.addIngreso(incomeToAdd)

        then:
        1 * movementAddService.saveMovement(_ as MovementToAdd) >> { List args ->
            def m = args[0] as MovementToAdd
            assert m.amount() == new BigDecimal("1000.00")
            assert m.type()   == MovementType.INGRESO.name()
        }
    }
}
```

---

## Entorno de Desarrollo

- **IDE:** IntelliJ IDEA con Gradle como build tool.
- **MapStruct + Lombok:** el procesamiento de anotaciones lo maneja Gradle. No hace falta crear ni configurar `.factorypath` manualmente — IntelliJ lo resuelve al sincronizar con Gradle.
- **Archivos de IDE:** nunca crear ni modificar `.project`, `.classpath`, `.settings/`, `.idea/`, `*.iml`, etc. Son gestionados por el IDE.

---
> Source: [matiasferrerovilas/api-movements](https://github.com/matiasferrerovilas/api-movements) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
