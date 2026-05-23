## mindhub

> MindHub es una plataforma integral de gestión sanitaria que integra múltiples módulos especializados para clínicas y profesionales de la salud.

# MindHub - Healthcare Management Platform

## Project Overview

MindHub es una plataforma integral de gestión sanitaria que integra múltiples módulos especializados para clínicas y profesionales de la salud.

## 🚀 ARQUITECTURA ACTUAL - DJANGO BACKEND + REACT CLEAN ARCHITECTURE

### 🏗️ **ARQUITECTURA COMPLETA**

```
┌─ Frontend React/Next.js ──── Vercel (https://mindhub.cloud) - CLEAN ARCHITECTURE
├─ API Proxy Routes ────────── Next.js (/api/*/django/)
├─ Django Backend ──────────── Django REST API (/backend-django/) - TODOS LOS MÓDULOS
├─ Auth Middleware ─────────── Supabase JWT validation
├─ Database ────────────────── Supabase PostgreSQL 
└─ Authentication ──────────── Supabase Auth
```

### 📁 **DOCUMENTACIÓN ARQUITECTÓNICA**

- **Arquitectura APIs**: `docs/architecture/MINDHUB_ARCHITECTURE_MASTER_COMPLETE.md` - 62+ endpoints documentados
- **Esquema Base de Datos**: `docs/architecture/SUPABASE_TABLES_REFERENCE.md` - Estructura exacta
- **Seguridad**: `docs/architecture/MINDHUB_SECURITY_ARCHITECTURE_MASTER.md` - Patrones de seguridad
- **Frontend**: React Clean Architecture (ver principios de desarrollo)
- **Backend**: Django REST Framework con todos los módulos migrados

### URLs de Producción (ACTUALES)

- **Frontend**: https://mindhub.cloud (Vercel)
- **Backend Django**: https://mindhub-django-backend.vercel.app
- **API Proxy**: https://mindhub.cloud/api/*/django/ (Next.js → Django)
- **Database**: Supabase PostgreSQL
- **Auth**: Supabase Auth

### Estado del Deployment

- ✅ **Backend completamente migrado a Django**
- ✅ **Node.js backend movido a legacy-backend**
- ✅ Django REST Framework con autenticación Supabase
- ✅ API proxy routes para integración seamless
- ✅ Sistema híbrido React + Django completamente funcional
- ✅ Todos los módulos (Expedix, Agenda, Resources) en Django
- ✅ Base de datos Supabase PostgreSQL integrada

### 🔐 **SISTEMA DE AUTENTICACIÓN - SUPABASE ÚNICAMENTE**

- **Proveedor**: Supabase Auth (https://supabase.com)
- **Frontend Auth**: `@supabase/auth-helpers-nextjs` con componentes React
- **Backend Auth**: Middleware Supabase en Django REST API
- **Usuario Principal**: Dr. Alejandro (dr_aleks_c@hotmail.com)
- **Funciones**:
  - ✅ Login/Logout automático
  - ✅ JWT tokens para APIs
  - ✅ Gestión de usuarios y sesiones
  - ✅ Row Level Security (RLS) en PostgreSQL
  - ✅ Integración nativa con Next.js
- **URLs de Auth**:
  - Sign In: https://mindhub.cloud/auth/sign-in
  - Sign Up: https://mindhub.cloud/auth/sign-up
  - Dashboard: https://mindhub.cloud/dashboard (post-login)

### Arquitectura del Sistema

```
MindHub-Pro/
├── mindhub/
│   ├── frontend/              # Next.js 14.2.30 + React 18 + TypeScript + Tailwind CSS
│   └── backend-django/        # Django REST API - Backend Principal
└── legacy-backend/            # Node.js backend (DEPRECATED - no usar)
```

### Stack Tecnológico Actual

**Frontend (React/Next.js):**
- Next.js 14.2.30 con App Router
- React 18 con TypeScript
- Tailwind CSS + shadcn/ui components
- Supabase client para auth y operaciones directas
- API proxy routes para Django integration

**Backend (Django REST):**
- Django 5.0.2 + Django REST Framework
- PostgreSQL vía Supabase connection
- Supabase JWT authentication middleware
- CORS configurado para frontend integration
- Modelos Django para todos los módulos (Expedix, Agenda, Resources, ClinimetrixPro)

## 🏥 Módulos Principales (Orden de Interconexión)

### 1. **Agenda** - Sistema de Citas y Programación

- **Frontend URL**: `/hubs/agenda`
- **Django API**: `https://mindhub-django-backend.vercel.app/api/agenda/`
- **Proxy API**: `https://mindhub.cloud/api/agenda/django/`
- **Estado**: ✅ **MIGRADO COMPLETAMENTE A DJANGO**
- **Funcionalidades**:
  - Programación de citas médicas - Django scheduling models
  - Gestión de horarios y disponibilidad - Django provider schedules
  - Drag & Drop para reprogramación - Frontend + Django integration
  - Notificaciones automáticas - Django signals
  - Lista de espera inteligente - Django waiting list system
  - Confirmación de citas - Django appointment workflow
  - Integración con Finance para cobros automáticos

### 2. **Expedix** - Gestión de Pacientes y Expedientes Médicos

- **Frontend URL**: `/hubs/expedix`
- **Django API**: `https://mindhub-django-backend.vercel.app/api/expedix/`
- **Proxy API**: `https://mindhub.cloud/api/expedix/django/`
- **Estado**: ✅ **MIGRADO COMPLETAMENTE A DJANGO**
- **Funcionalidades**:
  - Gestión completa de pacientes (CRUD) - Django models
  - Expedientes médicos digitales - Django serializers con 33+ campos
  - Sistema de consultas médicas - Django views con examen mental
  - Generación de recetas digitales - Django business logic
  - Historial médico completo - Django relationships
  - Portal de pacientes - Django authentication
  - Documentos médicos encriptados - Django security
  - Integración directa con Agenda para "INICIAR CONSULTA"

### 3. **ClinimetrixPro** - Sistema de Evaluaciones Psicométricas

- **Frontend URL**: `/hubs/clinimetrix`
- **Django API**: `https://mindhub-django-backend.vercel.app/api/clinimetrix/`
- **Proxy API**: `https://mindhub.cloud/api/clinimetrix/django/`
- **Estado**: ✅ **MIGRADO COMPLETAMENTE A DJANGO**
- **Funcionalidades**:
  - **29 escalas psicométricas migradas**: Desde PHQ-9 hasta escalas especializadas
  - **Motor de evaluación Django**: focused_take.html con Alpine.js CardBase
  - **Scoring inteligente**: Cálculos precisos y interpretaciones clínicas
  - **Integración con Expedix**: Resultados automáticamente asociados a pacientes
  - **Sistema de selección React**: UI/UX optimizada para selección de escalas
  - **Auto-guardado obligatorio**: Resultados permanentes en Supabase PostgreSQL

### 4. **FormX** - Generador de Formularios Médicos

- **Frontend URL**: `/hubs/formx`
- **Django API**: `https://mindhub-django-backend.vercel.app/api/formx/`
- **Proxy API**: `https://mindhub.cloud/api/formx/django/`
- **Estado**: ✅ **MIGRADO COMPLETAMENTE A DJANGO**
- **Funcionalidades**:
  - Creación de formularios personalizados con Django Forms
  - Templates médicos preconfigurrados
  - Formularios de registro de pacientes
  - Validación automática avanzada con Django
  - Integración con consultas médicas
  - Exportación de datos estructurados

### 5. **Finance** - Gestión Financiera y Facturación

- **Frontend URL**: `/hubs/finance`
- **Django API**: `https://mindhub-django-backend.vercel.app/api/finance/`
- **Proxy API**: `https://mindhub.cloud/api/finance/django/`
- **Estado**: ✅ **MIGRADO COMPLETAMENTE A DJANGO**
- **Funcionalidades**:
  - Sistema completo de facturación - Django finance models
  - Gestión de servicios y precios - Django pricing engine
  - Registro de ingresos automático - Django transaction system
  - Cortes de caja - Django cash register management
  - Integración con Agenda para cobros pendientes
  - Reportes financieros - Django reporting system
  - Multi-métodos de pago - Django payment processing

### 6. **FrontDesk** - Recepción y Gestión de Flujo

- **Frontend URL**: `/frontdesk` (NO bajo `/hubs/`)
- **Backend pattern**: ⚠️ **Direct-Supabase** (excepción al patrón Django) — las routes `/api/frontdesk/*` consultan Supabase vía `supabaseAdmin` desde Next.js, sin pasar por Django
- **Razón de la excepción**: módulo de lectura/agregación simple (stats del día, listas de citas, tareas pendientes) que no requiere lógica de negocio Django
- **Estado**: ✅ Funcional en producción con Direct-Supabase
- **Funcionalidades**:
  - Dashboard de recepción centralizado
  - Gestión de llegadas y esperas
  - Check-in automático de pacientes
  - Comunicación interna con profesionales
  - Gestión de documentos pendientes
  - Integración total con Agenda y Finance
  - Vista global de clínica multi-profesional

## Stack Tecnológico

### Frontend

- **Hosting**: Vercel - Auto deploy desde GitHub `main` branch
- **URL Producción**: https://mindhub.cloud
- **Framework**: Next.js 14.2.30 con App Router
- **UI Library**: React 18 con TypeScript
- **Styling**: Tailwind CSS + CSS Variables personalizadas
- **Componentes**: Sistema de componentes unificado
- **Estado**: Context API + useState/useEffect
- **Autenticación**: Supabase Auth - Sistema ÚNICO

### Backend Híbrido (Django + GraphQL Supabase)

- **Arquitectura**: Django REST Framework como backend principal + capa GraphQL Supabase para operaciones simples y fallbacks
- **Todos los módulos**: Agenda, Expedix, ClinimetrixPro, FormX, Finance, FrontDesk con backend Django
- **Base de Datos**: Supabase PostgreSQL - ÚNICO para todo el proyecto
- **ORM**: Django ORM conectado directamente a Supabase PostgreSQL
- **API Pattern primario**: Next.js Proxy Routes → Django REST → Supabase
- **API Pattern híbrido**: Hybrid services en `lib/*-hybrid-service.ts` (Django primary + Apollo/GraphQL fallback) — usar para Resources, Agenda settings, FormX y ClinimetrixPro
- **Apollo Client**: `GraphQLProvider` montado en `app/layout.tsx`; queries/mutations en `lib/apollo/`

### Infraestructura de Producción

- **Frontend + API Routes**: Vercel (https://mindhub.cloud)
- **Base de Datos**: Supabase PostgreSQL
- **Auth**: Supabase Auth
- **Django Backend**: https://mindhub-django-backend.vercel.app (producción)
- **Build**: Automático en deploy

## 🏗️ Principios de Desarrollo

### 🎯 **REACT CLEAN ARCHITECTURE (OBLIGATORIO)**

**Arquitectura en círculos concéntricos donde las capas internas no dependen de las externas:**

#### **1. Entidades (Core)**
**Propósito**: Encapsulan las reglas de negocio más críticas y universales
**En React**: Objetos o datos centrales con validaciones propias
```typescript
// Ejemplo: entities/Patient.ts
export class Patient {
  constructor(
    public id: string,
    public firstName: string,
    public lastName: string,
    public dateOfBirth: Date
  ) {
    this.validateAge();
  }
  
  private validateAge(): void {
    // Lógica de negocio pura
  }
}
```

#### **2. Casos de Uso (Application Business Rules)**
**Propósito**: Reglas de negocio específicas de la aplicación, orchestrando entidades
**En React**: Lógicas para interactuar con entidades
```typescript
// Ejemplo: usecases/CreatePatientUseCase.ts
export class CreatePatientUseCase {
  constructor(private patientRepository: PatientRepository) {}
  
  async execute(data: CreatePatientData): Promise<Patient> {
    // Orchestrar entidades y reglas de negocio
  }
}
```

#### **3. Adaptadores de Interfaz (Interface Adapters)**
**Propósito**: Traducen datos entre la lógica de dominio y la capa de implementación
**En React**: Comunicación con servicios externos, conversión de datos
```typescript
// Ejemplo: adapters/PatientApiAdapter.ts
export class PatientApiAdapter implements PatientRepository {
  async create(patient: Patient): Promise<Patient> {
    // Traducir entre dominio y API externa
  }
}
```

#### **4. Frameworks y Drivers (Externa)**
**Propósito**: Detalles de implementación (React, UI, base de datos, librerías)
**En React**: Componentes React, estado, servicios
```typescript
// Ejemplo: components/PatientForm.tsx
export const PatientForm: React.FC = () => {
  // UI pura, usa casos de uso vía dependency injection
}
```

### ✅ **BENEFICIOS DE CLEAN ARCHITECTURE EN REACT**

- **Independencia del Framework**: Lógica de negocio no ligada a React
- **Testabilidad**: Entidades y casos de uso fáciles de probar unitariamente
- **Mantenibilidad**: Separación de responsabilidades clara
- **Escalabilidad**: Estructura modular para crecimiento a largo plazo
- **Independencia de DB/UI**: Adaptable a diferentes tecnologías sin alterar el núcleo

### 🔄 **REGLAS FUNDAMENTALES**

1. **Inversión de Dependencias**: Capas internas NO conocen las externas
2. **Flujo de Dependencias**: Siempre hacia el interior (Entities ← Use Cases ← Adapters ← Frameworks)
3. **Abstracción**: Usar interfaces para desacoplar implementaciones
4. **Single Responsibility**: Cada capa tiene una responsabilidad específica

### 🎯 **APLICACIÓN EN MINDHUB**

```
Frontend React Clean Architecture:
├── entities/          # Patient, Appointment, Consultation (reglas de negocio puras)
├── usecases/          # CreatePatient, ScheduleAppointment (lógica de aplicación)  
├── adapters/          # ApiClients, DataTransformers (traducción de datos)
└── components/        # React Components (UI, framework específico)
```

### 📊 **Principios de Desarrollo Específicos**

#### **Gestión de Datos y Backend**

- **Base de Datos Principal**: Supabase PostgreSQL para todo el proyecto
- **Backend Principal**: Django REST Framework para lógica de negocio compleja, workflows, scoring, file ops
- **API Pattern primario**: Frontend → Next.js Proxy → Django → Supabase
- **API Pattern híbrido (permitido)**: Frontend → Apollo Client → Supabase GraphQL — solo para CRUD simple, dashboard stats, fallbacks de hybrid services. Ver `lib/*-hybrid-service.ts`
- **NO usar**: Conexiones directas Frontend → Supabase REST/SDK fuera de la capa Apollo o de los proxy routes
- **SIEMPRE**: Verificar arquitectura documentada antes de implementar cambios

#### **Arquitectura de APIs y Conectividad**

- **Documentación Principal**: `docs/architecture/MINDHUB_ARCHITECTURE_MASTER_COMPLETE.md` (62+ endpoints)
- **Esquema de DB**: `docs/architecture/SUPABASE_TABLES_REFERENCE.md` (estructura exacta)
- **Patrón de Autenticación**: Supabase JWT → Django middleware → PostgreSQL RLS
- **Sistema Dual**: `clinic_id` OR `workspace_id` (nunca ambos simultáneamente)

## 🔧 Principios de Implementación de Cambios

**SIEMPRE seguir este flujo al implementar cambios:**

1. **Verificar Arquitectura**: Consultar documentos de referencia primero
2. **Clean Architecture**: Implementar siguiendo capas (Entities → Use Cases → Adapters → Components)
3. **Backend Django**: Todos los módulos en Django REST, NO híbridos
4. **Frontend Completo**: No solo visual, sino funcionalmente integrado
5. **Database Pattern**: Usar estructura exacta documentada en docs/architecture/SUPABASE_TABLES_REFERENCE.md
6. **Testing**: Validar flujo completo Frontend → API → Backend → Database
7. **Integración**: Asegurar conexión entre módulos (Agenda ↔ Expedix ↔ Finance, etc.)

## 🔗 **INTERCONEXIÓN DE MÓDULOS**

### **📊 FLUJO DE INTEGRACIÓN COMPLETA**

```
Agenda → Expedix → ClinimetrixPro → FormX → Finance → FrontDesk
  ↓        ↓           ↓              ↓        ↓         ↓
  └────────┴───────────┴──────────────┴────────┴─────────┘
                   Django Backend Unificado
```

**Integraciones Críticas:**

1. **Agenda ↔ Finance**: Cobros automáticos al crear citas
2. **Agenda ↔ Expedix**: "INICIAR CONSULTA" crea consulta directa  
3. **Expedix ↔ ClinimetrixPro**: Evaluaciones asociadas automáticamente al expediente
4. **FormX ↔ Expedix**: Formularios personalizados en consultas
5. **FrontDesk ↔ Todos**: Dashboard centralizado de recepción
6. **Finance ↔ Todos**: Tracking financiero transversal

### **🎨 ESCALAS CLINIMETRIXPRO (29 DISPONIBLES)**

```
✅ Depresión: BDI-13, GDS-5/15/30, HDRS-17, MADRS, PHQ-9, RADS-2
✅ Ansiedad: GADI, HARS, STAI  
✅ Autismo/TEA: AQ-Adolescent, AQ-Child
✅ Trastornos Alimentarios: EAT-26
✅ Cognición: MOCA
✅ TOC: DY-BOCS, Y-BOCS
✅ Psicosis: PANSS
✅ Sueño: MOS Sleep Scale
✅ Tics: YGTSS
✅ Personalidad: IPDE-CIE10, IPDE-DSMIV
✅ Trauma: DTS
✅ Suicidalidad: SSS-V
```

---

## Recordatorios de Desarrollo

- No hagas commit ni push en github hasta que yo te lo pida. me puedes preguntar, pero no lo hagas sin que me autorice
- **Arquitectura actual**: Vercel Frontend + Django REST API + Supabase PostgreSQL
- **Backend principal**: Django REST Framework en `/mindhub/backend-django/`
- **Frontend**: React/Next.js en `/mindhub/frontend/` 
- **Autenticación**: 100% Supabase Auth con JWT validation en Django
- **Base de datos**: Supabase PostgreSQL para todo el proyecto

## ✅ **ESTADO ACTUAL DEL SISTEMA**

**Sistema Completamente Funcional en Producción:**

### **🚀 ÚLTIMOS AJUSTES IMPLEMENTADOS (Enero 2025)**

- ✅ **Dashboard restaurado**: Revertido a localStorage para funcionalidad completa
- ✅ **FormX production-ready**: API integration real completamente implementada
- ✅ **Migración localStorage completada**: Todos los módulos optimizados para producción
- ✅ **Settings comprehensivos**: Configuración completa para todos los módulos
- ✅ **Workflow Agenda → Consulta**: Restricciones críticas de base de datos resueltas
- ✅ **ClinimetrixPro optimizado**: Navegación settings, interpretaciones de escalas, soporte subscalas
- ✅ **Integración Expedix completa**: ClinimetrixPro totalmente integrado con optimizaciones
- ✅ **Suspense boundaries SSR**: Compatibilidad Next.js 14 en todas las páginas de navegación
- ✅ **Navegación URL consistente**: Implementada en todos los módulos MindHub
- ✅ **Errores críticos producción resueltos**: Restricciones database y display modal

### **📊 CARACTERÍSTICAS ACTUALES DEL SISTEMA**

**Backend Django Completamente Migrado:**
- ✅ **TODOS los módulos migrados**: Agenda, Expedix, ClinimetrixPro, FormX, Finance, FrontDesk
- ✅ **Node.js deprecado**: Backend anterior movido a `/legacy-backend/`
- ✅ **API unificada**: Django REST Framework con endpoints `/api/*` 
- ✅ **Proxy integration**: Frontend proxy routes hacia Django backend
- ✅ **Autenticación integrada**: Supabase JWT middleware en Django
- ✅ **Deploy completado**: Django backend en producción (Vercel)

**Frontend React Production-Ready:**
- ✅ **Clean Architecture**: Principios documentados y aplicados
- ✅ **localStorage optimized**: Sistema de almacenamiento local robusto para producción
- ✅ **Settings unificados**: Panel de configuración completo para todos los módulos
- ✅ **Navigation consistency**: URLs y navegación unificadas en toda la plataforma
- ✅ **SSR compatibility**: Suspense boundaries para Next.js 14
- ✅ **Production fixes**: Todos los errores críticos resueltos

**Integración y Flujos Completos:**
- ✅ **Agenda → Expedix workflow**: Flujo "INICIAR CONSULTA" completamente funcional
- ✅ **ClinimetrixPro integration**: 29 escalas completamente integradas con Expedix
- ✅ **FormX real API**: Integración de API real completamente implementada
- ✅ **Finance workflows**: Sistema completo de facturación y cobros
- ✅ **Settings comprehensivos**: Configuración granular para cada módulo
- ✅ **Database constraints**: Todas las restricciones críticas resueltas

### **🔧 PRINCIPIOS DE DESARROLLO ACTUALIZADOS**

**Reglas de Desarrollo Strictas:**
- ✅ **No soluciones temporales**: Los problemas se arreglan de fondo hasta que funcionen como deberían
- ✅ **Análisis profundo de errores**: Identificar raíz del problema, APIs, endpoints, autenticación
- ✅ **Scripts para configuración externa**: Cuando involucre settings de Supabase/Vercel
- ✅ **Información actualizada**: Verificar que datos de Vercel y Supabase estén actualizados
- ✅ **Testing completo**: Validar flujo completo Frontend → API → Backend → Database
- ✅ **Documentación actualizada**: Arquitectura y referencias siempre sincronizadas

**No hacer commit ni push sin autorización explícita del usuario.**

---
> Source: [dralejandroc/MINDHUB](https://github.com/dralejandroc/MINDHUB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
