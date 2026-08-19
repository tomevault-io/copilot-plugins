## abm

> This project builds scalable AI-powered business automation systems. The full blueprint lives in `objetivo-final/resumen_blueprint_automatizacion.md` — read it for deep context on philosophy, methodology, and case studies.

# AGENTS.md — ABM (Automatización de Negocios con IA)

## Project Purpose

This project builds scalable AI-powered business automation systems. The full blueprint lives in `objetivo-final/resumen_blueprint_automatizacion.md` — read it for deep context on philosophy, methodology, and case studies.

**Core mission:** Help businesses recover time, capture hot leads, and gain operational predictability through intelligent automation.

---

## Stack Tecnológico (Obligatorio)

Siempre usar este stack para todo desarrollo en este proyecto:

### Frontend
- React + Next.js + TypeScript
- Tailwind CSS

### Backend
- Node.js + TypeScript
- NestJS o Next.js (arquitecturas backend robustas y modulares)

### Base de Datos
- PostgreSQL (relacional principal)
- PG Vector (extensión esencial para sistemas RAG / bases de conocimiento de IA)
- Redis (caché y colas de mensajes)

### Infraestructura
- CI/CD con GitHub Actions
- Despliegue en Cloud (producción 24/7, nunca entornos locales inseguros)
- Reverse Proxy para seguridad de peticiones
- Rate Limiting obligatorio en todas las APIs

### Autenticación
- Auth estándar con roles de usuario

**NUNCA usar:** constructores visuales sin código ("Cloud Code") como solución final. Solo prototipos.

---

## Arquitectura de Automatizaciones

### Los 3 Pilares
Todo sistema se compone de: `Trigger → Cerebro (Reglas/IA) → Acción`

### Workflows vs Agentes
- **Workflows** → Procesos predecibles, lineales, deterministas (facturación, reportes, envíos masivos). Si se puede escribir como una receta inmutable, **usar Workflow**.
- **Agentes IA** → Procesos conversacionales con criterio (cualificación de leads, atención al cliente). Si requiere empatía, criterio o redacción contextualizada, **usar Agente**.
- **Híbrido (Recomendado):** Combinar reglas de negocio rígidas + IA para flexibilidad conversacional.

### Score-Based Routing
Los flujos de clasificación conversacional deben implementar un sistema cuantitativo de Score basado en recopilación de campos específicos. Enrutamiento dinámico solo cuando se alcance el umbral de calificación (ej. score >= 60).

---

## Filosofía de Negocio

### Qué vende realmente
1. **Tiempo recuperado** (horas de trabajo manual ahorradas por semana)
2. **Leads calientes que no se enfrían** (reducción de TTFR)
3. **Orden y previsibilidad** (procesos sistematizados)

### Modelo de Servicio
- No es ingreso pasivo. Es consultoría + desarrollo a medida + mantenimiento continuo.
- **NUNCA cobrar por horas** → cobrar tarifas fijas basadas en el valor del problema resuelto.
- **NUNCA regalar el mantenimiento** → todo sistema vivo requiere soporte continuo.

### Monetización
1. **Setup (Desarrollo inicial):** Precio anclado al valor del problema resuelto, nunca al tiempo de desarrollo. Ejemplo Strati: $8,000-$15,000 USD.
2. **Fee mensual recurrente:** Mantenimiento, mejoras, soporte operativo.

---

## Metodología de Diagnóstico

### 3 Semáforos de la Automatización
- 🟢 **Verde:** Automatizar inmediatamente — procesos repetitivos, alta frecuencia, reglas claras, sin criterio humano.
- 🟡 **Amarillo:** Automatizar con supervisión — la IA actúa como copiloto/asisistente.
- 🔴 **Rojo:** No tocar — procesos críticos de baja frecuencia, o integraciones sensibles con riesgo de multas.

### ROI
Siempre calcular la pérdida anual del cliente: `Horas perdidas/semana × Costo/hora × 52 semanas` y ofrecer la solución por una fracción de esa pérdida.

---

## Desarrollo — Directivas Técnicas

### Coding
- TypeScript strict mode en todo
- Sin `any` types — usar `unknown` y narrow con type guards
- `const` sobre `let`, nunca `var`
- Funciones pequeñas, responsabilidad única
- Early returns sobre anidamiento profundo
- Manejo explícito de errores, sin catches silenciosos

### Backend
- Type hints en todas las funciones
- Pydantic para schemas si se usa Python
- async/await para operaciones de BD y HTTP

### Frontend
- Interfaces para shapes de objetos, type aliases para unions
- Named exports sobre default exports
- Componentes: uno por archivo, nombre = nombre del archivo
- Props interface: `{ComponentName}Props`
- Sin estilos inline — usar Tailwind

### Seguridad
- Nunca hardcodear secrets, API keys o passwords
- Nunca loggear datos sensibles (tokens, passwords, PII)
- Validar todo input del usuario en boundaries de API
- Queries parametrizadas, sin concatenación de SQL

### Testing
- Testear comportamiento, no implementación
- Nombres descriptivos que expliquen el escenario
- Tests comentados se borran o se arreglan
- Mock de dependencias externas, no de funciones internas

### Git
- Conventional commits: `feat:`, `fix:`, `chore:`, etc.
- Sin secrets, credenciales ni .env en commits
- PRs describen QUÉ y POR QUÉ, no CÓMO

---

## Flujo de Desarrollo SDD (Spec-Driven Development)

```
Exploración → Propuesta → Resolución de Edge Cases → Specs → Design → Judgment Day → Tasks → Apply → Verify → Archive
```

### Fases
1. **Exploración:** La IA explora el proyecto, dependencias, y herramientas necesarias
2. **Propuesta:** Define alcance MVP, qué queda fuera, y límites claros
3. **Resolución de Edge Cases:** Preguntas específicas sobre decisiones de negocio críticas ANTES de codificar
4. **Specs:** Casos de prueba Given/When/Then para cada módulo
5. **Design:** Esquemas de BD, validaciones (Zod), contratos API (OpenAPI)
6. **Judgment Day:** Dos jueces IA en paralelo revisan el diseño de forma independiente. Corregir TODO warning crítico antes de continuar
7. **Tasks:** Backlog estructurado y secuencial
8. **Apply:** Implementación con progreso trackeado
9. **Verify:** Verificación contra specs y diseño original
10. **Archive:** Decisiones, specs modificadas, y reportes en registro persistente

### Herramientas de Validación
- **Zod:** Validación de schemas y contratos tipados
- **OpenAPI + codegen:** Sincronización de tipos FE↔BE
- **Vitest:** Pruebas unitarias e integración
- **Playwright:** Pruebas E2E

### Stack Confirmado
- **Backend:** NestJS + BullMQ + Redis (async workers, webhooks, procesamiento IA)
- **Frontend:** Next.js + Tailwind (dashboard, panel gerencial)
- **DB:** PostgreSQL + RLS (multitenant shared schema)
- **WhatsApp:** Meta WhatsApp Cloud API (oficial, producción 24/7)
- **IA:** Abstraction layer + OpenAI default
- **Real-time:** Socket.io (TTFR, leaderboard, distribución)
- **Infra:** Docker + GitHub Actions + Cloud 24/7

---

## Referencia

| Recurso | Ubicación |
|---------|-----------|
| Blueprint negocio | `objetivo-final/resumen_blueprint_automatizacion.md` |
| Blueprint arquitectura | `objetivo-final/blueprint_tanstack_sdd.md` |
| Skill registry | `.atl/skill-registry.md` |

---

*Este archivo es la fuente de verdad para el desarrollo en este proyecto. Cualquier IA o agente que trabaje aquí DEBE seguir estas directivas.*

---
> Source: [Jsoriano99/ABM](https://github.com/Jsoriano99/ABM) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
