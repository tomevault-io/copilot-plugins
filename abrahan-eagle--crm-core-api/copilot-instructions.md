## crm-core-api

> Backend API del sistema CRM especializado en gestión de préstamos comerciales, construido con NestJS 10 y TypeScript. El sistema gestiona el ciclo completo desde leads hasta comisiones, incluyendo contactos, empresas, aplicaciones de préstamo, bancos y ofertas.

# CRM Core API - Contexto para AI

## Descripción General
Backend API del sistema CRM especializado en gestión de préstamos comerciales, construido con NestJS 10 y TypeScript. El sistema gestiona el ciclo completo desde leads hasta comisiones, incluyendo contactos, empresas, aplicaciones de préstamo, bancos y ofertas.

## Arquitectura

### Stack Tecnológico
- **Framework**: NestJS 10.0.5
- **Lenguaje**: TypeScript 4.9.5
- **Base de Datos**: MongoDB 7.6.2 (Mongoose)
- **Autenticación**: Auth0 (express-oauth2-jwt-bearer)
- **Cloud Storage**: AWS S3
- **Email**: AWS SES
- **VoIP**: CloudTalk
- **Notificaciones**: NotificationAPI
- **Patrón**: Clean Architecture + CQRS

### Estructura de Carpetas
- `src/app/`: Capa de Aplicación (Command Handlers, Query Handlers, DTOs, Services)
- `src/domain/`: Capa de Dominio (Entities, Commands, Queries, Repository Interfaces, Events)
- `src/infra/`: Capa de Infraestructura (REST Controllers, MongoDB Adapters, External Adapters, DI)

### Patrones Arquitectónicos

#### 1. Clean Architecture / Hexagonal Architecture
- **Domain Layer (Inner)**: Entidades puras, interfaces de repositorios, lógica de negocio, independiente de frameworks
- **Application Layer (Middle)**: Command/Query Handlers, DTOs, Application Services, Event Handlers, orquesta casos de uso
- **Infrastructure Layer (Outer)**: REST Controllers, MongoDB implementations, External adapters, implementa detalles técnicos
- **Principio**: Las capas internas no dependen de las externas

#### 2. CQRS (Command Query Responsibility Segregation)
- **Commands**: Modifican estado (Create, Update, Delete) - `domain/*/commands/` y `app/*/commands/`
- **Queries**: Solo leen datos (Get, Search, List) - `domain/*/queries/` y `app/*/queries/`
- **Ventajas**: Optimización independiente, escalabilidad, separación clara

#### 3. Repository Pattern
- Interfaces en Domain Layer (`domain/*/repositories/`)
- Implementaciones en Infrastructure Layer (`infra/adapters/mongo/*/repositories/`)
- **Ventajas**: Independencia de BD, testabilidad, flexibilidad

#### 4. Event-Driven Architecture
- Domain Events en `domain/*/events/`
- Event Handlers en `app/*/events/`
- Ejemplo: `ApplicationAcceptedEvent` → Crea Commission automáticamente
- **Ventajas**: Desacoplamiento, extensibilidad, trazabilidad

#### 5. Dependency Injection (NestJS)
- Módulos NestJS por feature
- Providers registrados en módulos
- Guards y Filters globales
- **Ventajas**: Testabilidad, flexibilidad, mantenibilidad

#### 6. Module Pattern (Feature-Based)
- Organización por features (Application, Bank, Contact, etc.)
- Cada feature tiene su módulo NestJS
- **Ventajas**: Alta cohesión, bajo acoplamiento, escalabilidad

### Separación de Responsabilidades

#### Domain Layer (Núcleo)
- **Responsabilidad**: Lógica de negocio pura, independiente de tecnología
- **Componentes**: Entities, Value Objects, Domain Events, Repository Interfaces, Domain Services, Business Rules
- **Características**: No depende de frameworks, no depende de infraestructura, testeable sin infraestructura

#### Application Layer (Orquestación)
- **Responsabilidad**: Orquestar casos de uso, coordinar entre Domain y Infrastructure
- **Componentes**: Command Handlers, Query Handlers, DTOs, Application Services, Event Handlers
- **Características**: Depende de Domain, orquesta flujos, transforma DTOs ↔ Entities

#### Infrastructure Layer (Adaptadores)
- **Responsabilidad**: Implementar detalles técnicos, adaptar sistemas externos
- **Componentes**: REST Controllers, MongoDB Adapters, External Adapters, Mappers, Middlewares
- **Características**: Depende de Application y Domain, implementa interfaces, maneja detalles técnicos

#### Cross-Cutting Concerns
- **Componentes**: Guards (PermissionsGuard), Middlewares (Auth, Logging), Exception Filters, Interceptors
- **Características**: Aplicados globalmente, no pertenecen a una capa específica

### Decisiones de Diseño

#### Clean Architecture vs MVC
- **Decisión**: Clean Architecture
- **Razón**: Independencia, testabilidad, mantenibilidad
- **Trade-off**: Complejidad inicial mayor

#### CQRS vs CRUD
- **Decisión**: CQRS
- **Razón**: Optimización independiente, escalabilidad
- **Trade-off**: Más código, dos flujos diferentes

#### MongoDB vs PostgreSQL
- **Decisión**: MongoDB
- **Razón**: Flexibilidad, documentos anidados, escalabilidad horizontal
- **Trade-off**: Sin transacciones ACID complejas

#### Domain Events vs Direct Calls
- **Decisión**: Domain Events
- **Razón**: Desacoplamiento, extensibilidad
- **Trade-off**: Debugging más complejo

#### Repository Pattern vs Active Record
- **Decisión**: Repository Pattern
- **Razón**: Independencia, testabilidad
- **Trade-off**: Capa adicional, mappers necesarios

#### Feature-Based vs Layer-Based
- **Decisión**: Feature-Based
- **Razón**: Alta cohesión, bajo acoplamiento, escalabilidad
- **Trade-off**: Duplicación potencial

## Módulos Principales

### Application (Aplicaciones)
- **Propósito**: Gestión completa de solicitudes de préstamo comercial
- **Estados**: READY_TO_SEND → SENT → OFFERED → OFFER_ACCEPTED → COMPLETED
- **Comandos clave**: CreateApplication, AddNotificationsToApplication, AcceptOffer, CompleteApplication
- **Queries clave**: SearchApplications, GetApplicationById, GetBankNotifications, GetRecommendedBanks
- **Permisos**: CREATE_APPLICATION, READ_APPLICATION, LIST_APPLICATIONS, SEND_APPLICATION, etc.

### Bank (Bancos)
- **Propósito**: Gestión de instituciones financieras (lenders/brokers)
- **Operaciones clave**: createBank, updateBank, blacklistBank, sendEmailToBanks
- **Permisos**: CREATE_BANK, READ_BANK, LIST_BANKS, UPDATE_BANK, DELETE_BANK, SEND_EMAIL_TO_BANKS

### Contact (Contactos)
- **Propósito**: Gestión de personas físicas (miembros de empresas o independientes)
- **Validaciones**: Edad 21-99, máx 5 teléfonos/emails, máx 6 documentos
- **Operaciones clave**: createContact, updateContact, addFile, createNote
- **Permisos**: CREATE_CONTACT, READ_CONTACT, LIST_CONTACTS, UPDATE_CONTACT, DELETE_CONTACT, ADD_CONTACT_NOTE, DELETE_CONTACT_NOTE

### Company (Empresas)
- **Propósito**: Gestión de empresas que solicitan préstamos
- **Validaciones**: Máx 10 miembros, máx 4 documentos por tipo
- **Operaciones clave**: createCompany, updateCompany, addFile, createNote, transferCompany
- **Permisos**: CREATE_COMPANY, READ_COMPANY, LIST_COMPANIES, UPDATE_COMPANY, DELETE_COMPANY, TRANSFER_COMPANY, ADD_COMPANY_NOTE, DELETE_COMPANY_NOTE

### Commission (Comisiones)
- **Propósito**: Distribución de comisiones cuando se acepta una oferta
- **Estructura**: PSF + Commission, cada uno con distribución a múltiples usuarios
- **Estados**: DRAFT → PUBLISHED
- **Operaciones clave**: saveCommission, publishCommission
- **Permisos**: READ_COMMISSION, LIST_COMMISSIONS, UPDATE_COMMISSION, PUBLISH_COMMISSION

### Campaign (Campañas)
- **Propósito**: Campañas de marketing que generan leads
- **Estados**: STOPPED ↔ STARTED
- **Operaciones clave**: createCampaign, startCampaign, stopCampaign
- **Permisos**: LIST_CAMPAIGNS, CREATE_CAMPAIGN

### Lead (Prospectos)
- **Propósito**: Gestión de leads/prospectos antes de convertirlos en contactos
- **Operaciones clave**: createLead (desde CSV/Excel), searchProspects, addProspectNote
- **Flujo**: LeadGroup → Prospects → Contacts
- **Permisos**: CREATE_LEAD, READ_LEAD, LIST_LEADS, LIST_OWN_LEADS, ADD_PROSPECT_NOTE, TRANSFER_LEAD

### User (Usuarios)
- **Propósito**: Gestión de usuarios del sistema, roles y permisos
- **Operaciones clave**: createUser, updateUser, addRole, disableUser, makeACall
- **Permisos**: LIST_USER, CREATE_USER, UPDATE_USER, REQUEST_CALL, REQUEST_CUSTOM_CALL

## Flujos de Negocio Principales

### Flujo: Lead → Contact → Company → Application
1. Importar Leads (CSV/Excel) → LeadGroup con Prospects
2. Gestionar Prospects (llamadas, notas)
3. Convertir Prospect → Contact
4. Crear/Asociar Company → Asociar Contact como miembro
5. Crear Application desde Company
6. Enviar Application a Banks
7. Banks responden con Offers
8. Aceptar Offer → Commission creada automáticamente
9. Publicar Commission → Application completada

### Flujo: Crear y Enviar Aplicación
1. Seleccionar Company
2. Completar formulario (monto, producto, referral)
3. Subir documentos (bank statements, MTD, credit card, filled apps)
4. Crear aplicación (estado: READY_TO_SEND)
5. Establecer posición (1-5)
6. Seleccionar bancos destino
7. Enviar a bancos (estado: SENT)
8. Recibir ofertas (estado: OFFERED)
9. Aceptar oferta (estado: OFFER_ACCEPTED)
10. Commission creada automáticamente (DRAFT)
11. Completar aplicación (estado: COMPLETED)

## Sistema de Permisos

### Estructura
- Los permisos vienen en el JWT token de Auth0 (claim `permissions`)
- `PermissionsGuard` valida permisos antes de ejecutar endpoints
- Decorator `@RequiredPermissions()` especifica permisos requeridos

### Tipos de Permisos
- **"own" permissions**: Ver solo recursos propios (ej: LIST_OWN_APPLICATIONS)
- **"all" permissions**: Ver todos los recursos (ej: LIST_APPLICATIONS)
- **CRUD permissions**: CREATE, READ, UPDATE, DELETE por entidad
- **Action permissions**: SEND_APPLICATION, TRANSFER_APPLICATION, etc.
- **View Full permissions**: VIEW_FULL_SSN, VIEW_FULL_PHONE, VIEW_FULL_TAX_ID, VIEW_FULL_EMAIL, VIEW_FULL_NOTIFICATION

### Guards y Middlewares
- `PermissionsGuard` (Global): Valida permisos antes de ejecutar endpoint
- `validateAuthorizationToken`: Valida JWT token (Auth0)
- `CreateAuthContextMiddleware`: Crea contexto de autenticación
- `DecodeTokenMiddleware`: Decodifica token y extrae permisos

## Endpoints API Principales

### Applications
- `GET /v1/applications`: Buscar aplicaciones (con paginación y filtros)
- `GET /v1/applications/:id`: Obtener por ID
- `POST /v1/applications`: Crear aplicación (FormData: body + documents)
- `PUT /v1/applications/:id/notifications`: Enviar a bancos
- `GET /v1/applications/:id/notifications`: Obtener notificaciones
- `PUT /v1/applications/:id/notifications/:nId/accept/:offerId`: Aceptar oferta
- `PUT /v1/applications/:id/notifications/:nId/cancel/:offerId`: Cancelar oferta
- `PUT /v1/applications/:id/notifications/:nId/update/:offerId`: Actualizar oferta
- `PUT /v1/applications/:id/complete`: Completar aplicación
- `PATCH /v1/applications/:id/reject`: Rechazar aplicación
- `PUT /v1/applications/:id/substatus`: Actualizar subestado
- `PATCH /v1/applications/:id/position/:position`: Establecer posición (1-5)
- `DELETE /v1/applications/:id`: Eliminar aplicación
- `PUT /v1/applications/:id/transfer-to/:userId`: Transferir aplicación
- `GET /v1/applications/:id/recommended-banks`: Obtener bancos recomendados
- `GET /v1/last-application-period/:companyId`: Último período válido

**Drafts** (`/v1/drafts`):
- `GET /v1/drafts`: Buscar drafts
- `GET /v1/drafts/:id`: Obtener draft
- `POST /v1/drafts`: Crear draft
- `PUT /v1/drafts/:id`: Actualizar draft
- `PUT /v1/drafts/:id/publish`: Publicar draft (convierte a Application)
- `DELETE /v1/drafts/:id`: Eliminar draft
- `PUT /v1/drafts/:id/transfer-to/:userId`: Transferir draft
- **Permisos**: READ_DRAFT_APPLICATION, CREATE_DRAFT_APPLICATION, UPDATE_DRAFT_APPLICATION, PUBLISH_DRAFT_APPLICATION, DELETE_DRAFT_APPLICATION, TRANSFER_DRAFT

**Webhooks** (llamados por sistemas externos, no por frontend):
- `PUT /v1/applications/:id/send-to-banks`: Webhook para envío
- `PUT /v1/webhooks/applications/notification/reject`: Webhook de rechazo
- `POST /v1/webhooks/affiliate`: Webhook de affiliate
- `PUT /v1/webhooks/affiliate/consolidate`: Webhook de consolidación
- `GET /v1/campaigns/:campaign_id/send-next`: Webhook para enviar siguiente batch
- `POST /v1/campaigns/notification`: Webhook de notificación de campaña
- `POST /v1/campaigns/stop-all`: Webhook para detener todas las campañas

### Contacts
- `GET /v1/contacts`: Buscar contactos
- `GET /v1/contacts/:id`: Obtener contacto
- `GET /v1/contacts/ssn/:ssn`: Buscar por SSN
- `POST /v1/contacts`: Crear contacto
- `PUT /v1/contacts/:id`: Actualizar contacto
- `DELETE /v1/contacts/:id`: Eliminar contacto
- `POST /v1/contacts/:id/files`: Agregar archivo (FormData)
- `POST /v1/contacts/:id/notes`: Crear nota
- `PUT /v1/contacts/:id/transfer-to/:userId`: Transferir contacto

### Companies
- `GET /v1/companies`: Buscar companies
- `GET /v1/companies/:id`: Obtener company
- `POST /v1/companies`: Crear company
- `PUT /v1/companies/:id`: Actualizar company
- `DELETE /v1/companies/:id`: Eliminar company
- `POST /v1/companies/:id/files`: Agregar archivo (FormData)
- `POST /v1/companies/:id/notes`: Crear nota
- `PUT /v1/companies/:id/transfer-to/:userId`: Transferir company

### Banks
- `GET /v1/banks`: Buscar bancos
- `GET /v1/banks/:id`: Obtener banco
- `POST /v1/banks`: Crear banco
- `PUT /v1/banks/:id`: Actualizar banco
- `DELETE /v1/banks/:id`: Eliminar banco
- `GET /v1/banks/:id/offers`: Obtener ofertas
- `PUT /v1/banks/:id/blacklist`: Agregar a blacklist
- `POST /v1/banks/send-email`: Enviar email a bancos

### Commissions
- `GET /v1/commissions`: Buscar comisiones
- `GET /v1/commissions/:id`: Obtener comisión
- `PUT /v1/commissions/:id`: Guardar comisión (sin publicar)
- `PUT /v1/commissions/:id/publish`: Publicar comisión

### Campaigns
- `GET /v1/campaigns`: Listar campañas
- `POST /v1/campaigns`: Crear campaña
- `PUT /v1/campaigns/:id/start`: Iniciar campaña
- `PUT /v1/campaigns/:id/stop`: Detener campaña

### Leads
- `GET /v1/leads`: Buscar lead groups
- `GET /v1/leads/prospects`: Buscar prospects
- `POST /v1/leads`: Crear lead group (desde archivo CSV/Excel - FormData)
- `POST /v1/leads/prospects/:id/notes`: Agregar nota a prospect
- `PUT /v1/leads/:id/transfer-to/:userId`: Transferir lead group

### Users
- `GET /v1/users`: Buscar usuarios
- `POST /v1/users`: Crear usuario
- `PUT /v1/users/:id`: Actualizar usuario
- `POST /v1/users/make-a-call`: Realizar llamada

## Comunicación con Frontend

### Formato de Requests
- **JSON**: Para operaciones CRUD normales (GET, POST, PUT, PATCH, DELETE)
- **FormData**: Para requests con archivos (POST /v1/applications, POST /v1/contacts/:id/files, etc.)
  - Estructura esperada: `body` (JSON string) + `documents` (files array)
  - Frontend envía FormData con esta estructura

### Headers Esperados
- `Authorization`: JWT token de Auth0 (requerido para todas las rutas excepto públicas)
- `X-Tenant`: Tenant ID (inyectado por frontend HttpService)
- `Accept-Language`: Idioma preferido (inyectado por frontend HttpService)
- `Content-Type`: `application/json` (JSON) o `multipart/form-data` (FormData)

### Formato de Responses
- **Success**: Objeto JSON con datos
- **Error**: Objeto JSON con `statusCode`, `message`, `error`
- **Paginación**: `PaginatedResponse<T>` con `data`, `total`, `page`, `limit`

### Validaciones Coordinadas
- **Frontend**: Validaciones de UX (feedback inmediato al usuario)
- **Backend**: Validaciones de seguridad (nunca confiar solo en frontend)
- Algunas validaciones son redundantes por seguridad (defense in depth)
- Backend siempre valida permisos, estados, y reglas de negocio

### Domain Events y Side Effects
- Backend usa Domain Events para desacoplar side effects
- Ejemplo: Al aceptar oferta → `ApplicationAcceptedEvent` → Event Handler crea Commission automáticamente
- Frontend no necesita hacer requests adicionales, el backend maneja automáticamente
- Frontend recibe notificaciones en tiempo real vía NotificationAPI cuando hay cambios importantes

## Integraciones Externas

### Auth0
- Autenticación y autorización
- Permisos en JWT token (claim `permissions`)
- Middleware valida tokens automáticamente
- Frontend usa Auth0 SDK para autenticación

### MongoDB
- Base de datos principal
- Mongoose ODM
- Repositorios implementan interfaces de dominio

### AWS S3
- Almacenamiento de archivos
- S3MediaRepository implementa MediaRepository
- Multipart uploads para archivos grandes

### AWS SES
- Envío de emails
- SESMailerService para emails a bancos

### CloudTalk
- Sistema de llamadas telefónicas
- CloudTalkVoIPProviderRepository
- Endpoint `/v1/users/make-a-call`

### NotificationAPI
- Notificaciones push en tiempo real
- NotificationAPIRepository
- Envía notificaciones sobre cambios importantes

### Systeme
- Integración con sistema externo de contactos
- SystemeExternalContactsRepository
- Sincronización de contactos externos

## Validaciones y Reglas de Negocio

### Applications
- Monto: $1,000 - $20,000,000 (solo números enteros)
- Bank Statements: 4 períodos requeridos (calculados dinámicamente)
- Solo READY_TO_SEND puede enviarse a bancos
- Debe tener posición establecida (1-5) antes de enviar
- Mensaje a bancos: 15-800 caracteres (validado en backend)
- Al aceptar oferta: **Domain Event** `ApplicationAcceptedEvent` se dispara → Event Handler crea Commission automáticamente (DRAFT)
- No se permiten duplicados (misma company, mismo período)

### Contacts
- Edad: 21-99 años
- SSN: 9 dígitos (SSN o ITIN) - validación en backend
- Máximo 5 teléfonos
- Máximo 5 emails
- Máximo 6 documentos totales
- Máximo 4 documentos por tipo

### Companies
- Nombre: 2-100 caracteres (validación en backend)
- Máximo 10 miembros (mínimo 1)
- Máximo 4 documentos por tipo
- Tamaño máximo de archivo: 10MB

### Notes
- Niveles: INFO, WARNING, CRITICAL
- Algunos módulos requieren nota antes de salir

## Estados y Transiciones

### Application Status
```
READY_TO_SEND → SENT → OFFERED → OFFER_ACCEPTED → COMPLETED
                              ↓
                          REJECTED
```

### BankNotification Status
```
PENDING → SENT → OFFERED → ACCEPTED
                    ↓
                REJECTED
```

### Commission Status
```
DRAFT → PUBLISHED
```

### Campaign Status
```
STOPPED ↔ STARTED
```

## Convenciones de Código

### Naming
- Commands: `*Command` (CreateApplicationCommand, AcceptOfferCommand)
- Queries: `*Query` (GetApplicationByIdQuery, SearchApplicationsQuery)
- Handlers: `*CommandHandler`, `*QueryHandler`
- Entities: `*Entity` (ApplicationEntity, BankNotificationEntity)
- Repositories: `*Repository` (ApplicationRepository interface, MongoApplicationRepository implementation)
- Resources: `*Resource` (CreateApplicationResource, GetApplicationByIdResource)
- DTOs: `*Request`, `*Response` (CreateApplicationRequest, ApplicationResponse)

### Estructura de Archivos
- Commands: `domain/*/commands/` y `app/*/commands/`
- Queries: `domain/*/queries/` y `app/*/queries/`
- Entities: `domain/*/entities/`
- Repositories: `domain/*/repositories/` (interfaces) y `infra/adapters/mongo/*/repositories/` (implementaciones)
- Resources: `infra/adapters/rest/*/resources/`
- DTOs: `app/*/dtos/requests/` y `app/*/dtos/responses/`

### CQRS
- Commands modifican estado (Create, Update, Delete)
- Queries solo leen datos (Get, Search, List)
- Handlers separados para commands y queries

### Dependency Injection
- Módulos NestJS por feature
- Providers registrados en módulos
- Guards y Filters globales en AppModule

## Consideraciones Importantes

1. **Permisos**: Siempre verificar permisos antes de ejecutar operaciones
   - `PermissionsGuard` valida permisos globalmente
   - Decorator `@RequiredPermissions()` especifica permisos requeridos
   - Frontend también valida permisos para UX, pero backend siempre valida (seguridad)
2. **Archivos**: Usar FormData para requests con archivos, almacenar en S3
   - Frontend envía FormData con estructura: `body` (JSON string) + `documents` (files array)
   - Backend parsea FormData, valida archivos, y almacena en S3
3. **Estados**: Respetar transiciones de estado (no saltar estados inválidos)
   - Domain Entities validan transiciones de estado
   - Backend rechaza transiciones inválidas
   - Frontend debe reflejar estados correctamente
4. **Events**: Usar Domain Events para side effects (ej: crear Commission al aceptar oferta)
   - Domain Events desacoplan side effects
   - Event Handlers ejecutan acciones automáticamente
   - Frontend no necesita hacer requests adicionales
5. **Transferencias**: Las entidades pueden transferirse entre usuarios
   - Requiere permiso `TRANSFER_*` específico
   - Backend valida que usuario destino existe
6. **Own vs All**: Diferenciar entre permisos "own" y "all" en queries
   - Queries filtran resultados según permisos
   - "own" permissions: solo recursos del usuario actual
   - "all" permissions: todos los recursos
7. **Validaciones**: Validar reglas de negocio en Command Handlers
   - Frontend valida para UX (feedback inmediato)
   - Backend valida para seguridad (nunca confiar solo en frontend)
   - Algunas validaciones son redundantes por seguridad
8. **Mappers**: Usar mappers para convertir entre Domain Entities y MongoDB Documents
   - Domain Entities son independientes de MongoDB
   - Mappers convierten entre capas
9. **Multi-tenancy**: Al crear Application, se clona para todos los tenants
   - Frontend no necesita manejar esto explícitamente
   - Backend maneja multi-tenancy automáticamente
10. **Webhooks**: Algunos endpoints son webhooks (llamados por sistemas externos)
    - No son llamados por el frontend
    - Tienen autenticación diferente (WebhookAuthMiddleware)
    - Rutas definidas en AppModule.WEBHOOK_ROUTES

## Contexto de Negocio

### Propósito del Sistema
Facilitar la gestión completa del proceso de préstamos comerciales, desde la generación de leads hasta el cierre y distribución de comisiones.

### Usuarios Típicos
- **Agentes/Brokers**: Gestionan sus propios leads, contacts, companies y applications
- **Supervisores**: Supervisan todo el equipo
- **Administradores**: Gestionan bancos, usuarios, permisos
- **Marketing**: Crea campañas y genera leads

### Flujo Típico de Trabajo
1. Marketing crea campaña → genera leads
2. Agente trabaja leads → convierte a contacts/companies
3. Agente crea application desde company
4. Agente envía application a múltiples banks
5. Banks responden con offers
6. Agente acepta offer → commission creada automáticamente
7. Administrador configura y publica commission
8. Application se completa

## Análisis Profundo de Módulos

### Application Module
**Problema que Resuelve**: Centralización de solicitudes de préstamo, coordinación con múltiples bancos, gestión compleja de documentos, trazabilidad completa.

**Flujo Completo - Crear Application**:
1. Validar permisos CREATE_APPLICATION
2. Validar Company existe
3. Calcular períodos (última aplicación o estándar)
4. Validar documentos (4 bank statements requeridos, otros opcionales)
5. Validar no duplicados (mismo período)
6. Crear Application entity (estado: READY_TO_SEND)
7. Clonar para todos los tenants
8. Guardar en MongoDB (transacción)
9. Guardar archivos en S3
10. Retornar 201 Created

**Reglas de Negocio**:
- Monto: $1,000 - $20,000,000
- Bank Statements: Exactamente 4 períodos (calculados dinámicamente)
- No duplicados en mismo período para misma company
- Solo READY_TO_SEND puede enviarse a bancos
- Al aceptar oferta: crea Commission automáticamente (Domain Event)

**Integraciones**:
- Company Module: Valida Company existe, obtiene última aplicación
- Bank Module: Envía a bancos, recibe ofertas
- Commission Module: Crea Commission automáticamente (ApplicationAcceptedEvent)
- AWS S3: Guarda documentos
- NotificationAPI: Notificaciones de cambios

### Contact Module
**Problema que Resuelve**: Gestión centralizada de personas, información completa (SSN, documentos, notas, llamadas), relación flexible con Companies.

**Validaciones**:
- Edad: 21-99 años
- SSN: 9 dígitos (SSN o ITIN)
- Teléfonos: Máximo 5
- Emails: Máximo 5
- Documentos: Máximo 6 totales, 4 por tipo

**Integraciones**:
- Company Module: Contact puede ser miembro de múltiples Companies
- Lead Module: Prospect puede convertirse en Contact
- AWS S3: Guarda documentos

### Company Module
**Problema que Resuelve**: Gestión de empresas cliente, gestión de miembros, relación con Applications.

**Validaciones**:
- Nombre: 2-100 caracteres
- Miembros: 1-10 (mínimo 1, máximo 10)
- Documentos: Máximo 4 por tipo

**Integraciones**:
- Contact Module: Company tiene Members (Contacts)
- Application Module: Company puede tener múltiples Applications

### Bank Module
**Problema que Resuelve**: Catálogo centralizado de bancos, constraints y criterios, blacklist, gestión de ofertas.

**Estructura**:
- Bank: nombre, tipo (LENDER/BROKER), manager, status, constraints, blacklist
- Constraints: amount min/max, industries, territories
- Sistema calcula bancos recomendados basado en constraints

**Reglas**:
- Bank puede estar ACTIVE o INACTIVE
- Bank puede estar en blacklist
- Constraints determinan qué aplicaciones puede recibir
- Bank en blacklist no aparece en recomendados

**Integraciones**:
- Application Module: Bank recibe Applications (vía BankNotifications)
- AWS SES: Envío de emails a bancos

### Commission Module
**Problema que Resuelve**: Distribución automática de comisiones, configuración flexible, trazabilidad.

**Flujo**:
1. Application acepta oferta → ApplicationAcceptedEvent
2. Event Handler crea Commission automáticamente (DRAFT)
3. Administrador configura distribución PSF y Commission
4. Publica: estado → PUBLISHED (no puede editarse más)

**Estructura**:
- Commission: PSF + Commission, cada uno con distribución a múltiples usuarios
- Estados: DRAFT → PUBLISHED

**Integraciones**:
- Application Module: Commission se crea automáticamente desde Application (Domain Event)
- User Module: Distribution items apuntan a Users

### Campaign Module
**Problema que Resuelve**: Generación automática de leads, gestión de campañas, asignación automática.

**Estructura**:
- Campaign: sender, subject, message, contacts, jobId
- Estado: jobId !== null = STARTED, jobId === null = STOPPED

**Flujo**:
1. Crear Campaign (STOPPED)
2. Iniciar: Scheduler Service crea job
3. Job genera leads automáticamente
4. Leads se asignan a agentes automáticamente

**Integraciones**:
- Scheduler Service: Crea jobs para generar leads
- Lead Module: Campaign genera LeadGroups

### Lead Module
**Problema que Resuelve**: Importación masiva de prospectos, gestión de prospectos, seguimiento de interacciones, conversión a Contactos.

**Estructura**:
- LeadGroup: fileName, totalProspects, ownerId
- Prospect: company, name, email, phone, notes, callHistory, followUpCall

**Flujo - Importar Leads**:
1. Subir archivo CSV/Excel
2. Parser lee filas
3. Para cada fila: crear Prospect entity
4. Validar Prospect (nombre, teléfono, email)
5. Crear LeadGroup con todos los Prospects
6. Asignar owner al usuario actual
7. Guardar en MongoDB

**Integraciones**:
- Contact Module: Prospect puede convertirse en Contact
- Campaign Module: Campaign genera LeadGroups
- VoIP: Llamadas desde Prospects se registran en callHistory

### User Module
**Problema que Resuelve**: Gestión de usuarios, sistema de llamadas, gestión de affiliates.

**Flujo - Realizar Llamada**:
1. Validar permisos REQUEST_CALL
2. Validar teléfono válido
3. Llamar a CloudTalk API
4. Registrar CallLog en Prospect/Contact
5. Retornar 200 OK

**Integraciones**:
- CloudTalk (VoIP): Realiza llamadas telefónicas
- Auth0: Sincroniza usuarios y permisos

## Convenciones de Commits y Pull Requests

### Conventional Commits

Este proyecto sigue la especificación **Conventional Commits** para estructurar los mensajes de commit.

#### Estructura
```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

#### Tipos Principales
- `feat`: Nueva funcionalidad (MINOR en SemVer)
- `fix`: Corrección de bug (PATCH en SemVer)
- `docs`: Cambios en documentación
- `refactor`: Refactorización sin cambiar funcionalidad
- `test`: Añadir o modificar tests
- `build`: Cambios en build o dependencias
- `chore`: Tareas de mantenimiento

#### Scope (Opcional)
Ejemplos: `application`, `contact`, `company`, `bank`, `commission`, `lead`, `auth`, `infra`, `domain`

#### Breaking Changes
Indicar con `!` después del tipo/scope o en footer:
```
feat(application)!: change status enum

BREAKING CHANGE: Application status values have been updated
```

#### Ejemplos
- `feat(application): add cancel offer functionality`
- `fix(application): correct status update on offer cancellation`
- `docs: add Conventional Commits guide to README`
- `refactor(application): simplify status transition logic`
- `test(application): add unit tests for cancelOffer`

#### Pull Requests
- Título debe seguir formato Conventional Commits
- Descripción debe incluir: Qué, Por qué, Cómo, Testing, Breaking Changes (si aplica)
- Checklist: código sigue convenciones, tests pasando, documentación actualizada, sin errores de linting

### Reglas del Proyecto

1. **Idioma**: Todo el código y comentarios en **inglés**
2. **Clean Code**: Seguir principios de Clean Code (nombres descriptivos, funciones pequeñas, código autodocumentado, DRY)
3. **Conventional Commits**: Usar para todos los commits (ver sección anterior)
4. **No commits directos a main**: Todo debe pasar por Pull Requests con revisión y aprobación
5. **Sugerencias**: Bienvenidas, pero en el momento adecuado (code reviews, planificación)

### Patrones y Principios

- **Result Pattern**: Manejo de errores funcional
- **Clean Architecture**: Domain, Application, Infrastructure layers
- **CQRS Pattern**: Commands (escritura) y Queries (lectura) separados
- **SOLID Principles**: Aplicados en todo el código
- **MongoDB Aggregations**: Para consultas complejas

### Flujo de Trabajo

1. Crear branch desde `main`: `git checkout -b feat/feature-name`
2. Desarrollar siguiendo Clean Code
3. Commits con Conventional Commits
4. Push y crear PR
5. Esperar code review y aprobación
6. Merge a `main`

---
> Source: [Abrahan-Eagle/crm-core-api](https://github.com/Abrahan-Eagle/crm-core-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
