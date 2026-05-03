## gondolapp

> > Este archivo define cómo dividir el trabajo entre agentes especializados.

# AGENTS.md — Instrucciones para agentes de Claude Code

> Este archivo define cómo dividir el trabajo entre agentes especializados.
> Cada agente tiene un scope claro, una rama de Git propia y responsabilidades definidas.
> El orquestador (vos) integra el trabajo de todos y resuelve conflictos.

---

## Arquitectura de agentes

```
Orquestador (Manuel)
├── AGENTE 1: Backend & Database     → rama: agent/backend
├── AGENTE 2: App Gondolero          → rama: agent/gondolero
├── AGENTE 3: Paneles Web            → rama: agent/paneles
└── AGENTE 4: Admin & Config         → rama: agent/admin
```

Cada agente **lee CLAUDE.md completo** antes de arrancar.
Cada agente trabaja en **su rama** y nunca toca archivos de otras ramas.
La integración ocurre en `main` — solo el orquestador hace merge.

---

## AGENTE 1 — Backend & Database

**Scope:** Todo lo que no tiene interfaz visual. Base de datos, tipos, utilidades, middleware.

**Archivos de su propiedad:**
```
supabase/migrations/         # SQL de migraciones
supabase/seed.sql            # Datos de prueba
types/database.ts            # Tipos generados de Supabase
types/index.ts               # Tipos globales del negocio
lib/supabase/client.ts       # Supabase browser client
lib/supabase/server.ts       # Supabase server client
lib/supabase/middleware.ts   # Auth middleware
lib/utils.ts                 # Helpers (Haversine, formatters, etc.)
lib/validations/             # Schemas Zod
middleware.ts                # Next.js middleware (auth routing)
```

**Tareas del MVP en orden:**
1. Crear todas las migraciones SQL (tablas, RLS, índices)
2. Configurar los dos clientes de Supabase (browser y server)
3. Crear el middleware de autenticación y routing por rol
4. Generar los tipos TypeScript desde el schema
5. Implementar helpers: cálculo de distancia Haversine, formateo de puntos/tokens
6. Crear schemas Zod para todos los formularios del MVP
7. Crear seed.sql con datos de prueba para el piloto

---

## AGENTE 2 — App Gondolero (Mobile PWA)

**Scope:** Todo lo que ve y usa el gondolero en su celular. Mobile-first, offline-capable.

---

## AGENTE 3 — Paneles Web (Distribuidora + Marca)

**Scope:** Los paneles desktop para distribuidoras y marcas. Data-dense, desktop-first.

---

## AGENTE 4 — Auth & Admin

**Scope:** Autenticación compartida y panel de administración interno de GondolApp.

---

*Última actualización: Marzo 2026*

---
> Source: [manueldo-md/gondolapp](https://github.com/manueldo-md/gondolapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
