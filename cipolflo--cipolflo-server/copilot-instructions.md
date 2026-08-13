## cipolflo-server

> Este archivo describe las convenciones, patrones y restricciones del proyecto para que cualquier agente de IA pueda contribuir correctamente sin romper la arquitectura existente.

# AGENTS.md — Guía para agentes de IA

Este archivo describe las convenciones, patrones y restricciones del proyecto para que cualquier agente de IA pueda contribuir correctamente sin romper la arquitectura existente.

---

## Qué es este proyecto

Backend Spring Boot para el sistema de gestión del Club CIPOLFLO. Expone una API REST que maneja clientes (particulares y socios), servicios ofrecidos, reservas y finanzas (ingresos/egresos).

- **Lenguaje:** Java 21
- **Framework:** Spring Boot 3.x
- **Build:** Gradle con Kotlin DSL (`build.gradle.kts`)
- **Base de datos:** PostgreSQL
- **Puerto:** 8080

---

## Estructura del proyecto

```
src/main/java/com/cipolflo/server/
├── shared/                     ← Clases transversales. No depende de ningún módulo.
├── clientes/                   ← Módulo de clientes y socios
├── servicios/                  ← Módulo de servicios del club
├── reservas/                   ← Módulo de reservas
├── finanzas/                   ← Módulo de ingresos y egresos
└── integraciones/              ← Integración con servicios externos (no es un dominio de negocio)
```

Cada módulo de dominio (`clientes`, `servicios`, `reservas`, `finanzas`) tiene esta
estructura interna fija:

```
{modulo}/
├── domain/
│   └── enums/
├── repository/
├── service/
├── dto/
└── controller/
```

No crear carpetas fuera de este esquema salvo acuerdo explícito.

### `integraciones/` — excepción acordada al esquema de módulo

`integraciones/` no sigue la estructura `domain/repository/service/dto/controller` de los
módulos de negocio: es la capa de integración con servicios externos (Telegram, y a
futuro otros canales de mensajería o IA). Se parte en **núcleo agnóstico + adaptador**:

```
integraciones/
├── mensajeria/      ← núcleo: no sabe qué es Telegram. Solo conoce sus propios
│                       puertos (CanalMensajeria, RegistroDestinatarios) y tipos
│                       genéricos (destinatarioId: String). Acá viven las tools de IA,
│                       el asistente (Spring AI) y la memoria conversacional.
└── telegram/        ← adaptador: conoce el formato de update de Telegram, implementa
                        los puertos de mensajeria/, expone el webhook.
```

Regla: si se agrega un canal nuevo (ej. WhatsApp), se escribe un adaptador nuevo
(`integraciones/whatsapp/`) implementando los mismos puertos de `mensajeria/` — el núcleo
no cambia. Ver [`docs/telegram-bot-arquitectura.md`](docs/telegram-bot-arquitectura.md)
para el detalle completo.

---

## Convenciones de código

### Lombok

Todas las entidades usan Lombok. El patrón estándar en entidades es:

```java
@Getter
@Setter
@NoArgsConstructor
```

- Nunca escribir getters/setters a mano si Lombok los puede generar.
- Cuando un campo no debe ser modificable desde fuera de la entidad, usar `@Setter(AccessLevel.NONE)` en ese campo específico (ver `Reserva.java`).
- Cuando el constructor público no debe existir (entidad con factory method), usar `@NoArgsConstructor(access = AccessLevel.PROTECTED)`.

### Entidades JPA

- Todas las entidades extienden `AuditableEntity` (provee `createdAt` y `updatedAt` automáticos vía JPA Auditing).
- Los enums se persisten siempre como `@Enumerated(EnumType.STRING)`, nunca `ORDINAL`.
- Las IDs son `Long` con `@GeneratedValue(strategy = GenerationType.IDENTITY)`.
- Los campos requeridos llevan `@Column(nullable = false)`.
- Los campos únicos llevan `@Column(unique = true, nullable = false)`.

### Herencia de entidades

Hay dos jerarquías con `SINGLE_TABLE`:

| Tabla     | Discriminador | Subtipos              |
| --------- | ------------- | --------------------- |
| `cliente` | `tipo`        | `PARTICULAR`, `SOCIO` |
| `finanza` | `tipo`        | `INGRESO`, `EGRESO`   |

Si se agrega un nuevo subtipo, debe anotarse con `@DiscriminatorValue("NOMBRE")` y extender la clase abstracta correspondiente. No crear una tabla nueva salvo que la herencia cambie de estrategia.

### Enums compartidos vs de módulo

- Enums usados por **más de un módulo** van en `shared/enums/` (ej. `FormaPago`, `Procedencia`).
- Enums usados por **un solo módulo** van en `{modulo}/domain/enums/` (ej. `EstadoReserva`, `EstadoSocio`).

### Nombres

- Clases en español siguiendo el dominio del negocio (igual que el código existente).
- Interfaces de servicio: `I{Nombre}Service`.
- Implementaciones: `{Nombre}Service`.
- DTOs: `{Nombre}RequestDto` y `{Nombre}ResponseDto`.
- Tests: `{Nombre}Test` para dominio, `{Nombre}ServiceTest` para servicios.

---

## Patrones de dominio — no romper

### Factory method en `Reserva`

`Reserva` tiene constructor protegido. La única forma válida de crear una reserva es:

```java
Reserva r = Reserva.crear(clienteId, servicioId, procedencia, fechaEntrada, fechaSalida, requiereDocumentacion);
```

No agregar un constructor público ni cambiar la visibilidad del existente.

### Máquina de estados de `Reserva`

Las transiciones de estado son:

```
PENDIENTE → CONFIRMADA → EN_CURSO → FINALIZADA
```

`CONFIRMADA` se alcanza automáticamente cuando se registran pago (`confirmarPago`) **y** documentación (`recibirDocumentacion`), estando en `PENDIENTE`. No modificar `estado` directamente desde el exterior — usar los métodos de la entidad.

Si se agrega un nuevo estado, actualizar `esTransicionValida()` en `Reserva.java`.

### Lógica de negocio en `Socio`

`Socio` expone dos métodos de negocio:

- `pasarAInactivoPorMorosidad()`: fuerza estado `INACTIVO`. Lo usa `InactivacionSociosService`,
  que recalcula desde `pago_cuota` cuántos meses completos adeuda el socio (sin contador
  persistido) y llama a este método al llegar a 3.
- `darDeBaja()`: fuerza estado `DE_BAJA`.

No modificar `estado` directamente desde el servicio — usar estos métodos.

### `AuditableEntity`

`createdAt` y `updatedAt` son gestionados por Spring Data JPA Auditing. No asignarlos manualmente ni agregar setters.

---

## Capa de servicio

- Cada módulo tiene una interfaz `I{Modulo}Service` y su implementación `{Modulo}Service`.
- La implementación está anotada con `@Service`.
- Toda la lógica de negocio que no sea pura lógica de dominio va en el servicio.
- Los controllers no acceden al repositorio directamente, siempre pasan por el servicio.

---

## Controllers y API

- Base de URLs: `/api/v1/{modulo}` (ej. `/api/v1/clientes`, `/api/v1/reservas`).
- Devuelven DTOs, nunca entidades directamente.
- Usar `@RestController` y `@RequestMapping` en la clase.
- Todo parámetro DTO debe anotarse con `@Valid` para activar las validaciones de Jakarta (ej. `public ResponseEntity<?> crear(@Valid @RequestBody ClienteRequestDto dto)`).
- El endpoint de health check (`GET /api/health`) está en `shared/HealthController.java` — no duplicarlo.

---

## DTOs

- Los DTOs están en `{modulo}/dto/`.
- `RequestDto` recibe datos del cliente HTTP. Sus campos deben llevar anotaciones de Jakarta Validation (`@NotNull`, `@NotBlank`, `@Min`, `@Max`, `@Size`, `@Email`, etc.). No usar validación manual en constructores.
- `ResponseDto` es lo que se serializa en la respuesta. Todo DTO de respuesta debe implementar la marker interface `com.cipolflo.server.shared.dto.ResponseDto`.
- No exponer entidades JPA en respuestas HTTP.
- El mapeo entidad ↔ DTO es responsabilidad del servicio (o de un mapper si se introduce uno).
- `PaginationMapper.toPageResponse()` solo acepta `Page<T extends ResponseDto>`. Siempre convertir la entidad a DTO antes de paginar.

---

## Tests

### Ubicación

Los tests **espejan exactamente** la estructura de `src/main` dentro de `src/test`:

```
src/main/java/com/cipolflo/server/clientes/service/ClienteService.java
src/test/java/com/cipolflo/server/clientes/service/ClienteServiceTest.java

src/main/java/com/cipolflo/server/reservas/domain/Reserva.java
src/test/java/com/cipolflo/server/reservas/domain/ReservaTest.java
```

Regla: el test de `Foo.java` vive en el mismo paquete que `Foo.java`, pero bajo `src/test`.

### Qué testear y cómo

| Capa              | Tipo de test                        | Notas                                                                        |
| ----------------- | ----------------------------------- | ---------------------------------------------------------------------------- |
| `domain/`         | Unitario (sin Spring)               | Instanciar directamente, sin mocks. Foco en invariantes y lógica de negocio. |
| `service/`        | Unitario con mocks                  | Mockear el repositorio. Verificar llamadas y transformaciones.               |
| `controller/`     | Integración parcial (`@WebMvcTest`) | Solo si hay lógica de mapeo o validación no trivial en el controller.        |
| Contexto completo | `@SpringBootTest`                   | Solo para smoke tests o tests end-to-end puntuales.                          |

### Prioridades

1. Tests de dominio primero: `Reserva`, `Socio` tienen lógica de negocio crítica que debe estar cubierta.
2. Luego tests de servicio.
3. Los tests de controller son opcionales hasta que los endpoints estén estabilizados.

---

## Migraciones de base de datos (Liquibase)

El esquema se versiona con Liquibase. El changelog raíz es `src/main/resources/db/changelog/db.changelog-master.yaml`, que **solo referencia changelogs, no archivos SQL sueltos** (salvo el esquema inicial `001_initial_schema.sql`).

### Estructura

Cada ticket que requiera cambios de esquema crea su propia carpeta bajo `db/changelog/migrations/{TICKET}/`:

```
db/changelog/
├── db.changelog-master.yaml              ← referencia el changelog de cada carpeta
└── migrations/
    ├── 001_initial_schema.sql            ← esquema inicial
    └── DEV-117/
        ├── db.changelog-DEV-117.yaml     ← changelog general: incluye los SQL de la carpeta
        └── alter_tipo_fechas_columnas_reserva.sql
```

### Reglas

1. **Una carpeta por ticket**: `migrations/{TICKET}/` (ej. `DEV-117/`).
2. **Archivos SQL con nombre descriptivo** de lo que hacen (ej. `alter_tipo_fechas_columnas_reserva.sql`), no numéricos.
3. **Changelog general de la carpeta**: cada carpeta tiene un `db.changelog-{TICKET}.yaml` que incluye, en orden, todos los SQL agregados en esa carpeta.
4. **El master referencia el changelog de la carpeta**, nunca los SQL directamente.
5. Los SQL usan `--liquibase formatted sql`, con un `--changeset cipolflo:{TICKET}-descripcion` por cambio y su `--rollback` correspondiente.

Para agregar otra migración al mismo ticket: crear el `.sql` en la carpeta del ticket y sumarlo al `db.changelog-{TICKET}.yaml` de esa carpeta. El master no se toca.

---

## Credenciales y variables de entorno

Las credenciales de base de datos **nunca** van hardcodeadas en archivos versionados.

- `docker-compose.yml` lee las variables desde `.env` (Docker Compose lo carga automáticamente).
- `application.properties` usa `${DB_USERNAME}` y `${DB_PASSWORD}` — Spring Boot las toma del entorno del proceso.
- `.env` está en `.gitignore`. El archivo versionado es `.env.example`, que contiene la plantilla con valores de ejemplo.

Si necesitás agregar una nueva variable de entorno, agregala en `.env.example` con un valor de ejemplo y documentá su propósito ahí mismo.

---

## Lo que no hacer

- No agregar lógica de negocio en los controllers.
- No acceder al repositorio desde el controller.
- No exponer entidades JPA como respuesta HTTP.
- No modificar `AuditableEntity` para agregar campos funcionales — es solo para auditoría.
- No agregar constructores públicos a entidades que usan factory methods (`Reserva`).
- No cambiar la estrategia de herencia (`SINGLE_TABLE`) sin evaluar el impacto en la base de datos.
- No crear paquetes nuevos fuera de la estructura `domain/repository/service/dto/controller` dentro de un módulo sin razón justificada.
- No usar `@Enumerated(EnumType.ORDINAL)`.
- No hardcodear credenciales en ningún archivo versionado — siempre usar variables de entorno.

---

## Cómo agregar una nueva funcionalidad

Seguir este orden:

1. **Dominio** — entidad o campo en `{modulo}/domain/`. Enums en `{modulo}/domain/enums/`.
2. **Repository** — queries custom si se necesitan, en `{modulo}/repository/`.
3. **DTOs** — definir campos en `{modulo}/dto/`.
4. **Servicio** — método en `I{Modulo}Service` e implementación en `{Modulo}Service`.
5. **Controller** — endpoint en `{Modulo}Controller`.
6. **Test** — en `src/test/.../.../{modulo}/` al mismo nivel que la clase testeada.

Si el concepto es transversal a varios módulos → va en `shared/`.

---

## Cómo agregar un módulo nuevo

1. Crear carpeta bajo `src/main/java/com/cipolflo/server/{nuevo-modulo}/`.
2. Crear las subcarpetas: `domain/`, `domain/enums/`, `repository/`, `service/`, `dto/`, `controller/`.
3. Crear la entidad extendiendo `AuditableEntity`.
4. Crear `I{Nuevo}Service` y `{Nuevo}Service`.
5. Crear `{Nuevo}RequestDto` y `{Nuevo}ResponseDto`.
6. Crear `{Nuevo}Controller` con `@RequestMapping("/api/v1/{nuevo-modulo}")`.
7. Crear la carpeta espejo en `src/test/java/com/cipolflo/server/{nuevo-modulo}/`.

---
> Source: [CIPOLFLO/cipolflo-server](https://github.com/CIPOLFLO/cipolflo-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
