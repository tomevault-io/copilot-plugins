## realmeet

> Este proyecto es una plataforma SaaS de agendamiento profesional para profesionales de salud, abogados y futuras categorías de servicios profesionales.

# AGENTS.md

# Contexto general del proyecto

Este proyecto es una plataforma SaaS de agendamiento profesional para profesionales de salud, abogados y futuras categorías de servicios profesionales.

La plataforma permite que:
- Profesionales se registren, configuren su perfil, disponibilidad, modalidad de atención y calendario.
- Usuarios/clientes/pacientes se registren, busquen profesionales y agenden horas según disponibilidad.
- Administradores gestionen usuarios, profesionales, categorías, especialidades, reservas y configuración general.
- El sistema envíe correos de confirmación y quede preparado para integraciones con Google Meet, Zoom, WhatsApp, pagos e IA.

El objetivo es construir primero un MVP sólido, simple y escalable. No implementar funcionalidades avanzadas antes de tener la base estable.

---

# Stack obligatorio

Backend:
- Python 3.12+
- FastAPI
- SQLAlchemy 2
- Alembic
- PostgreSQL
- Pydantic
- JWT para autenticación
- Passlib o bcrypt para password hashing
- APScheduler para tareas simples iniciales
- SMTP para correos
- Arquitectura preparada para Celery en el futuro, pero no usar Celery en la Fase 1 salvo que se indique explícitamente

Frontend:
- React
- Vite
- TypeScript
- Tailwind CSS
- React Router
- Axios o TanStack Query
- Zustand o Context API para estado global

Infraestructura:
- Docker
- Docker Compose
- Variables de entorno
- `.env.example`
- README con instrucciones claras

Base de datos:
- PostgreSQL
- Migraciones con Alembic
- Seeds iniciales para entorno local

---

# Principios de arquitectura

El proyecto debe priorizar:
- Código limpio
- Separación de responsabilidades
- Modularidad
- Escalabilidad
- Seguridad desde el inicio
- Facilidad de despliegue
- Buenas prácticas DevOps
- Bajo costo operativo inicial

No implementar lógica compleja directamente en endpoints.

La estructura del backend debe separar:
- routers / api
- schemas
- models
- services
- repositories
- core
- db
- integrations
- emails
- utils

La estructura del frontend debe separar:
- pages
- components
- layouts
- api
- features
- routes
- store
- types
- utils

---

# Fases del producto

## Fase 1 - MVP obligatorio

Implementar solamente:

- Autenticación
- Roles: admin, professional, client
- Registro de clientes
- Registro de profesionales
- Perfil profesional
- Categorías
- Especialidades
- Disponibilidad semanal del profesional
- Bloqueo manual de horarios
- Búsqueda de profesionales
- Vista de perfil público del profesional
- Reserva de hora
- Historial de reservas
- Dashboard profesional
- Métricas básicas
- Backoffice administrador mínimo
- Envío de correos
- Meeting provider mock

No implementar todavía:
- Google Meet real
- Zoom real
- WhatsApp real
- Pagos reales
- IA
- Firma digital
- Facturación
- Suscripciones

Esas funcionalidades deben quedar preparadas mediante interfaces o servicios desacoplados.

---

# Roles y permisos

Existen tres roles principales:

## Admin

Puede:
- Ver todo
- Gestionar usuarios
- Gestionar profesionales
- Gestionar categorías
- Gestionar especialidades
- Gestionar reservas
- Ver métricas globales
- Activar/suspender usuarios
- Activar/suspender profesionales
- Modificar configuración general

## Professional

Puede:
- Ver y editar su perfil profesional
- Configurar disponibilidad
- Bloquear fechas
- Ver sus reservas
- Confirmar reservas
- Cancelar reservas
- Completar reservas
- Ver historial de atenciones
- Ver métricas propias
- Agregar notas privadas a una atención

No puede:
- Ver reservas de otros profesionales
- Ver datos administrativos globales
- Modificar categorías o especialidades

## Client

Puede:
- Ver y editar su perfil
- Buscar profesionales
- Ver disponibilidad
- Agendar horas
- Cancelar sus propias reservas futuras
- Ver su historial de reservas

No puede:
- Ver reservas de otros clientes
- Ver notas privadas del profesional
- Acceder al backoffice

---

# Reglas de negocio críticas

- Un profesional no puede tener dos reservas en el mismo horario.
- Un cliente no puede reservar el mismo bloque dos veces.
- La disponibilidad se calcula en base a:
  - reglas semanales del profesional
  - duración de sesión
  - bloqueos manuales
  - reservas existentes
- Las notas privadas del profesional nunca deben ser visibles para el cliente.
- Solo el admin puede ver información global.
- El profesional solo puede ver información asociada a su propio perfil.
- El cliente solo puede ver sus propias reservas.
- Toda acción sensible debe validar permisos por rol.
- Toda reserva debe quedar registrada con historial de cambios.
- No eliminar datos sensibles físicamente si existe historial asociado; preferir soft delete o estados.

---

# Estados de reserva

Usar estos estados:

- pending
- confirmed
- cancelled
- completed
- no_show

No inventar otros estados sin actualizar documentación, schemas, modelos y frontend.

---

# Entidades principales

Mantener como base estas entidades:

- User
- ProfessionalProfile
- ClientProfile
- Category
- Specialty
- ProfessionalSpecialty
- AvailabilityRule
- AvailabilityBlock
- Appointment
- AppointmentHistory
- AuditLog
- SystemSetting

Si se agregan nuevas entidades, justificar brevemente en README o comentario técnico.

---

# Seguridad

Aplicar siempre:

- Password hashing
- JWT con expiración
- Validación con Pydantic
- Variables de entorno
- CORS configurable
- No hardcodear secretos
- No commitear `.env`
- No registrar passwords, tokens ni datos sensibles en logs
- Control de acceso por rol en backend
- Protección de rutas en frontend
- Manejo consistente de errores

Considerar que el sistema puede manejar datos sensibles de salud o información legal. Por lo tanto, minimizar exposición de datos y aplicar principio de menor privilegio.

---

# Licencias

Usar únicamente dependencias open source compatibles con uso comercial.

Preferir licencias:
- MIT
- BSD-3-Clause
- Apache-2.0
- PostgreSQL License

Evitar dependencias:
- GPL
- AGPL
- Licencias restrictivas
- SDKs comerciales no necesarios

Si se agrega una dependencia importante, documentarla en README con su propósito.

---

# Integraciones

Las integraciones deben diseñarse de forma desacoplada.

## Meetings

Crear una interfaz o clase base tipo:

- MeetingProvider

Implementaciones esperadas:

- MockMeetingProvider
- GoogleMeetProvider, preparado pero no funcional en Fase 1
- ZoomProvider, preparado pero no funcional en Fase 1

En Fase 1 usar `MockMeetingProvider`.

La reserva debe poder guardar:
- meeting_provider
- meeting_url
- external_meeting_id
- calendar_event_id

## Emails

Crear servicio desacoplado de email.

Debe soportar:
- envío real vía SMTP si existen variables configuradas
- modo desarrollo/log si no hay SMTP configurado

Eventos mínimos:
- reserva creada
- reserva confirmada
- reserva cancelada
- reserva completada

---

# Métricas

## Profesional

Mostrar:
- reservas de hoy
- próximas reservas
- atenciones del mes
- total histórico de atenciones
- clientes únicos
- cancelaciones
- tasa de cancelación
- ingresos estimados si existe precio
- especialidades más solicitadas

## Admin

Mostrar:
- total usuarios
- total profesionales
- total clientes
- total reservas
- reservas por estado
- reservas por categoría
- profesionales más activos
- crecimiento mensual

---

# Estilo de código backend

- Usar typing explícito.
- Usar Pydantic schemas para entrada y salida.
- No devolver modelos ORM directamente si exponen campos sensibles.
- Usar services para lógica de negocio.
- Usar repositories para acceso a datos cuando corresponda.
- Usar dependencias de FastAPI para DB session y usuario autenticado.
- Manejar errores con HTTPException o excepciones propias.
- Mantener endpoints delgados.
- Escribir funciones pequeñas y testeables.
- No duplicar lógica de permisos.

---

# Estilo de código frontend

- Usar TypeScript estricto.
- Crear componentes reutilizables.
- Mantener páginas limpias y apoyadas en componentes.
- Separar llamadas API en `/src/api`.
- Separar tipos en `/src/types`.
- No hardcodear URLs de backend.
- Usar variables de entorno.
- Mostrar loading, error y empty states.
- Proteger rutas según rol.
- Mantener UI sobria, profesional y responsive.

---

# UX/UI

La plataforma debe sentirse:
- profesional
- confiable
- moderna
- sobria
- útil tanto para salud como para abogados

Evitar un diseño exclusivamente médico o exclusivamente legal.

Usar:
- cards
- tablas limpias
- badges de estado
- calendario o vista semanal
- formularios claros
- dashboard con métricas visuales

---

# Comandos esperados

El proyecto debe poder levantarse con:

```bash
docker compose up --build
```

Backend local:

```bash
cd backend
uvicorn app.main:app --reload
```

Frontend local:

```bash
cd frontend
npm install
npm run dev
```

Migraciones:

```bash
cd backend
alembic upgrade head
```

---

# Seeds iniciales

Crear seed con:

- Admin demo
- Cliente demo
- Profesional demo
- Categorías:
  - Salud
  - Legal
- Especialidades salud:
  - Psicología
  - Medicina General
  - Nutrición
- Especialidades legal:
  - Derecho Laboral
  - Derecho Familiar
  - Derecho Civil

Documentar credenciales demo en README.

---

# Criterios de aceptación

Una tarea se considera terminada solo si:

- El backend levanta sin errores.
- El frontend levanta sin errores.
- Las migraciones corren correctamente.
- El seed inicial funciona.
- Existe README actualizado.
- Existen variables en `.env.example`.
- No hay secretos hardcodeados.
- Los endpoints principales están protegidos por rol.
- La funcionalidad implementada puede probarse manualmente.
- No se rompió funcionalidad anterior.

---

# Forma de trabajo esperada para Codex

Antes de implementar cambios grandes:

1. Leer este archivo.
2. Revisar estructura actual del proyecto.
3. Proponer plan breve.
4. Implementar en pasos pequeños.
5. Mantener cambios coherentes con la arquitectura.
6. Actualizar README si cambia configuración, comandos o variables.
7. No introducir dependencias innecesarias.
8. No mezclar muchas funcionalidades en un solo cambio.

---

# Decisiones por defecto

Si hay dudas, tomar estas decisiones:

- Preferir simplicidad sobre complejidad.
- Preferir MVP funcional sobre arquitectura sobreingenierizada.
- Preferir PostgreSQL desde el inicio.
- Preferir APScheduler inicialmente.
- Preferir interfaces mock para integraciones externas.
- Preferir Docker Compose para desarrollo local.
- Preferir diseño modular para permitir migrar a microservicios en el futuro, pero mantener monolito modular en Fase 1.
- Preferir seguridad y privacidad sobre velocidad cuando existan datos sensibles.
- Preferir dependencias open source permisivas.

---

# Prohibiciones

No hacer lo siguiente salvo instrucción explícita:

- No implementar pagos en Fase 1.
- No implementar WhatsApp real en Fase 1.
- No implementar Google Meet real en Fase 1.
- No implementar Zoom real en Fase 1.
- No agregar dependencias GPL/AGPL.
- No guardar tokens o secretos en código.
- No crear lógica de negocio directamente en componentes React.
- No crear lógica de negocio pesada dentro de routers FastAPI.
- No saltarse migraciones de base de datos.
- No usar SQLite para esta plataforma salvo pruebas muy puntuales.
- No exponer notas privadas del profesional al cliente.
- No permitir acceso administrativo a usuarios no admin.

---
> Source: [fraactal/RealMeet](https://github.com/fraactal/RealMeet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
