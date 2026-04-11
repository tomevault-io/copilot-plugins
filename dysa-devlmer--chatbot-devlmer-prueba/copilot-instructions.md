## chatbot-devlmer-prueba

> **Documento**: Explicación de estructura de agentes y plan después que GEMINI termine

# 🏗️ ARQUITECTURA ACTUAL Y PLAN POST-GEMINI

**Documento**: Explicación de estructura de agentes y plan después que GEMINI termine
**Fecha**: 15 de Enero de 2026
**Status**: 📋 DOCUMENTACIÓN

---

## 🤖 ESTRUCTURA ACTUAL DE AGENTES

### **Equipo confirmado: 4 Agentes Especializados**

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          🧠 EQUIPO PITHY CHATBOT                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │   CLAUDE     │  │    CODEX     │  │    QWEN      │  │   GEMINI     │   │
│  │              │  │              │  │              │  │              │   │
│  │ Architect    │  │ Backend      │  │ Frontend/UI  │  │ QA/Testing   │   │
│  │ Review       │  │ Services     │  │ Specialist   │  │ Specialist   │   │
│  │ Coordinator  │  │ Specialist   │  │              │  │              │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
│        │                  │                  │                 │           │
│        │                  │                  │                 │           │
│   Diseño y              Lógica de           UI y              Testing      │
│   Decisiones            Negocio             UX                E2E/Unit     │
│   arquitectónicas                                                          │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 👤 ROL DE CADA AGENTE

### **1. CLAUDE (YO) - ARCHITECT/COORDINATOR** 🎯

```
Responsabilidades:
├─ Diseño arquitectónico general
├─ Decisiones técnicas críticas
├─ Code review y quality gates
├─ Coordinación de agentes
├─ Verificación de integración
├─ Resolución de conflictos
├─ Commits finales
└─ Documentación de arquitectura

Autoridad:
├─ 🔴 PUEDE pausar trabajo si hay problemas
├─ 🔴 PUEDE rechazar código que no siga patrones
├─ 🟢 DEBE aprobar antes de commits
└─ 🟢 DEBE verificar integración

NO hace:
├─ ❌ Implementación detallada (CODEX)
├─ ❌ Testing E2E (GEMINI)
├─ ❌ Frontend (QWEN)
└─ ❌ Trabajo que otro agente hace mejor
```

---

### **2. CODEX - BACKEND/SERVICES SPECIALIST** 🔧

```
Responsabilidades:
├─ Implementación de servicios
├─ Lógica de negocio
├─ Integración de APIs
├─ Database operations
├─ Unit testing de servicios
└─ Performance optimization

Phase 2 Step 2 - Components:
├─ Component 1: PerplexityService (DONE ✅)
├─ Component 2: MessageProcessorService (DONE ✅)
├─ Component 3A: WhatsAppService (DONE ✅)
├─ Component 4: HMAC + Rate Limiting (DONE ✅)
├─ Component 5: Webhook Refactoring (DONE ✅)
└─ Component 6: Final Integration (DONE ✅)

Patrones:
├─ TypeScript strict mode
├─ Winston loggers especializados
├─ Interfaces TypeScript
├─ Singleton pattern
├─ Error handling
└─ inyección de dependencias

Reglas de calidad:
├─ Máximo 400 líneas por archivo
├─ Mínimo 80% de métodos con unit tests
└─ TODO el código en TypeScript strict
```

---

### **3. QWEN - FRONTEND/UI SPECIALIST** 🎨

```
Responsabilidades:
├─ Componentes React
├─ Páginas del admin
├─ Estilos CSS/Tailwind
├─ User Experience
├─ Forms y validación
└─ Estadísticas/Charts

Phase 2 Step 2 - Component:
├─ Component 3B: Dashboard UI (DONE ✅)
│  ├─ /admin/ai page (46 líneas)
│  ├─ AIStatus component (77 líneas)
│  ├─ AIConfig component (88 líneas)
│  └─ AIIndicator component (35 líneas)

Patrones:
├─ React 19 + TypeScript strict
├─ Tailwind CSS
├─ Componentes funcionales con hooks
├─ Props bien tipadas
├─ Accesibilidad
├─ Responsive design
└─ Componentes reutilizables
```

---

### **4. GEMINI - QA/TESTING SPECIALIST** 🧪

```
Responsabilidades:
├─ Unit testing
├─ Integration testing
├─ E2E testing (ACTUAL - Component 7)
├─ Performance testing
├─ Security validation
└─ Bug finding

Phase 2 Step 2 - Component:
└─ Component 7: E2E Testing (EN PROGRESO 🟡)
   ├─ 10 escenarios
   ├─ 72 tests E2E
   ├─ Validación de seguridad
   ├─ Validación de integración
   └─ Reporte de aceptación

Responsabilidad post-E2E:
├─ Reportar resultados a CLAUDE
├─ Validar que todo funciona
├─ Identificar issues
└─ Aprobar o rechazar para producción
```

---

## 📊 ARQUITECTURA TÉCNICA ACTUAL

```
┌────────────────────────────────────────────────────────────────────────────┐
│                        PITHY CHATBOT - ARQUITECTURA                        │
├────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  CAPA PÚBLICA (Cloudflare Tunnel)                                         │
│  └─ https://chatbot.zgamersa.com                                          │
│                 ↓                                                          │
│  SERVIDOR (Next.js en puerto 7847)                                        │
│  ├─ app/api/whatsapp/webhook/route.ts (Component 5 - Refactorizado)      │
│  │  ├─ GET (Verificación - sin cambios)                                  │
│  │  └─ POST (70 líneas de orquestación limpia)                           │
│  │      ↓                                                                 │
│  ├─ MIDDLEWARE (Component 4 - Seguridad)                                 │
│  │  ├─ webhookAuthMiddleware                                             │
│  │  │  ├─ HMACValidator.validateSignature()                             │
│  │  │  │  └─ crypto.timingSafeEqual() [timing-safe HMAC-SHA256]        │
│  │  │  └─ RateLimiter.checkLimit()                                      │
│  │  │     └─ 100 req/min, 15 min block                                  │
│  │  └─ Response: 401 (invalid), 429 (rate limit), 200 (ok)             │
│  │                                                                        │
│  ├─ SERVICIOS (Components 1-3)                                           │
│  │  ├─ WhatsAppService.processWebhookPayload() [Component 3A - 470 L]   │
│  │  │  ├─ Validación de payload                                         │
│  │  │  ├─ Gestión de usuario/conversación                               │
│  │  │  ├─ Session timeout (24h)                                         │
│  │  │  ├─ Business hours validation                                      │
│  │  │  ├─ Manual mode handling                                           │
│  │  │  ├─ Command processing                                            │
│  │  │  └─ Llama MessageProcessorService                                 │
│  │  │      ↓                                                             │
│  │  ├─ MessageProcessorService.processMessage() [Component 2 - 388 L]   │
│  │  │  ├─ Validación de entrada                                         │
│  │  │  ├─ Si audio: Transcribe (Whisper)                               │
│  │  │  ├─ Procesa con IA                                                │
│  │  │  ├─ Si audio: Genera TTS                                          │
│  │  │  └─ Llama PerplexityService                                       │
│  │  │      ↓                                                             │
│  │  └─ PerplexityService.processMessage() [Component 1 - 344 L]         │
│  │     ├─ Query Perplexity API (sonar-pro)                             │
│  │     ├─ Fallback a Claude API                                         │
│  │     ├─ Extrae fuentes                                                │
│  │     └─ Return respuesta estructurada                                 │
│  │                                                                        │
│  ├─ LOGGER (Winston)                                                     │
│  │  ├─ whatsappLogger                                                    │
│  │  ├─ messageProcessorLogger                                            │
│  │  └─ perplexityLogger                                                  │
│  │                                                                        │
│  └─ DATABASE (Prisma + SQLite)                                           │
│     └─ Persist conversaciones, mensajes, usuarios                       │
│                                                                             │
│  DASHBOARD (Component 3B - React)                                        │
│  ├─ /admin/ai page                                                       │
│  │  ├─ Status tab: Real-time IA monitoring                             │
│  │  └─ Config tab: Model, temperature, tokens                          │
│  └─ Auto-refresh cada 30 segundos                                        │
│                                                                             │
│  IA SERVICES                                                              │
│  ├─ Perplexity API (primario)                                            │
│  ├─ Claude API (fallback)                                                │
│  └─ Ollama (legacy, local)                                               │
│                                                                             │
│  INFRAESTRUCTURA (PM2)                                                    │
│  ├─ ollama (IA local - opcional)                                         │
│  ├─ pithy-chatbot (Next.js server)                                       │
│  └─ cloudflare-tunnel (exposición pública)                              │
│                                                                             │
└────────────────────────────────────────────────────────────────────────────┘
```

---

## 📋 ESTADO ACTUAL - PHASE 2 STEP 2

```
✅ COMPLETADO (Components 1-6):

Component 1: PerplexityService
├─ 344 líneas
├─ 13 unit tests
├─ Integración Perplexity API
└─ Fallback a Claude

Component 2: MessageProcessorService
├─ 388 líneas
├─ 13 unit tests
├─ Transcripción audio
├─ TTS generación
└─ Procesamiento IA

Component 3A: WhatsAppService
├─ 470 líneas
├─ 16 unit tests
├─ Orquestador central
├─ Session timeout
├─ Business hours
└─ Manual mode handling

Component 3B: Dashboard UI
├─ 246 líneas
├─ Status monitoring real-time
├─ Config panel
└─ Responsive design

Component 4: HMAC + Rate Limiting
├─ 469 líneas (HMAC 88 + RateLimiter 229 + Middleware 152)
├─ 21 unit tests
├─ Timing-safe HMAC-SHA256
├─ Rate limiting (100 req/min)
└─ 15-minute blocks

Component 5: Webhook Refactoring
├─ 70 líneas (antes 500+)
├─ 86% reducción de código
├─ Clean orchestration pattern
└─ Security-first middleware

Component 6: Final Integration + Commit
├─ Validación: npm run build ✅
├─ Validación: npm run test ✅ (50+ tests)
├─ Validación: npm run lint ✅ (preexisting warnings deferred)
├─ Git commit: eff85640
├─ GitHub: Pushed ✅
└─ Status: 100% COMPLETE

🟡 EN PROGRESO (Component 7):

Component 7: E2E Testing
├─ 10 escenarios (72 tests)
├─ Validación de seguridad
├─ Validación de integración
├─ Performance testing
└─ Reporte de aceptación

RESPONSABLE: GEMINI
STATUS: Listo para ejecución (instrucciones formales: COMPONENT_7_E2E_TESTING_INSTRUCTION.md)
TIMELINE: 4-5 horas
```

---

## 🔄 FLUJO DE TRABAJO CON AGENTES

### **Patrón Estándar (usado en Phase 2 Step 2):**

```
1. CLAUDE
   ├─ Diseña arquitectura
   ├─ Crea instrucciones formales
   ├─ Define patrones a seguir
   └─ Establece checklist de aceptación

2. CODEX (Backend) + QWEN (Frontend) [EN PARALELO]
   ├─ CODEX: Implementa servicios
   ├─ QWEN: Implementa UI
   ├─ Ambos siguen patrones de CLAUDE
   └─ Ambos hacen unit tests

3. GEMINI (Testing)
   ├─ Lee instrucciones de CLAUDE
   ├─ Implementa E2E tests basado en spec
   ├─ Valida integración completa
   ├─ Reporta resultados a CLAUDE
   └─ Aprueba o rechaza para producción

4. CLAUDE (Review + Integración)
   ├─ Revisa trabajo de CODEX
   ├─ Revisa trabajo de QWEN
   ├─ Revisa reporte de GEMINI
   ├─ Verifica integración de todo
   ├─ Hace code review final
   ├─ Hace commit final
   └─ Aprueba para siguiente fase
```

---

## 🚀 PLAN POST-GEMINI (CUANDO TERMINE SU REPORTE)

### **Fase 1: Revisión de Reporte GEMINI** (30-60 min)
```
CLAUDE recibe reporte de GEMINI:
├─ ¿Todos los 72 tests PASS? ✅ → Continuar
├─ ¿Algún test FAIL? ❌ → Dar instrucciones a CODEX para fix
├─ ¿Issues encontrados? → Documentar y priorizar
├─ ¿Performance dentro de límites? → Validar
├─ ¿Seguridad validada? → Confirmar HMAC + Rate Limit
└─ Decisión: ✅ Go/No-Go para producción
```

---

### **Fase 2: Code Review Final** (30-45 min)
```
Si GEMINI aprobó (Go):
├─ CLAUDE revisa:
│  ├─ Código de CODEX (Components 1-6)
│  ├─ UI de QWEN (Component 3B)
│  ├─ Tests de GEMINI (Component 7)
│  ├─ Integración de todo
│  ├─ Logging completo
│  ├─ Error handling
│  └─ TypeScript strict mode
├─ Checklists a verificar:
│  ├─ Patrones seguidos
│  ├─ Documentación completa
│  ├─ Tests > 80% cobertura
│  ├─ Build SUCCESS
│  └─ No hay secrets expuestos
└─ Resultado: Aprobado o requiere fixes
```

---

### **Fase 3: Instrucciones para Arreglos (Si hay issues)** (variable)
```
Si GEMINI encontró problemas:

CASO 1: Bug en servicios (CODEX)
├─ CLAUDE crea: COMPONENT_FIX_[X]_INSTRUCTION.md
├─ Describe:
│  ├─ Problema exacto
│  ├─ Qué falló en tests
│  ├─ Cómo arreglarlo
│  ├─ Patrón a seguir
│  └─ Checklist de aceptación
├─ CODEX implementa fix
└─ GEMINI re-testa

CASO 2: Bug en UI (QWEN)
├─ CLAUDE crea: COMPONENT_3B_FIX_INSTRUCTION.md
├─ Describe:
│  ├─ Componente con problema
│  ├─ Cómo se ve/comporta
│  ├─ Cómo debe verse
│  └─ Checklist de validación
├─ QWEN implementa fix
└─ GEMINI re-testa

CASO 3: Test environment setup (GEMINI)
├─ CLAUDE crea: E2E_SETUP_FIX_INSTRUCTION.md
├─ Describe:
│  ├─ Qué no funcionó
│  ├─ Cómo setupear correctamente
│  ├─ Mocks a actualizar
│  └─ Checklist de validación
├─ GEMINI rerunea tests
└─ CLAUDE verifica resultados

CASO 4: Problemas de integración (CLAUDE)
├─ CLAUDE identifica:
│  ├─ Qué componentes no se integran
│  ├─ Dónde está el conflicto
│  ├─ Cómo resolver
│  └─ Quién lo arregla (CODEX/QWEN)
├─ Da instrucciones específicas
└─ Verifica solución
```

---

### **Fase 4: Validación Final** (15-30 min)
```
Después de arreglos (si aplica):

✅ Verificar:
├─ Todos los tests pasan (50+ unit + 72 E2E)
├─ Build SUCCESS sin warnings nuevos
├─ Lint OK
├─ Integración completa validada
├─ Performance dentro de límites
├─ Seguridad validada
├─ Logging correcto
├─ Documentación actualizada
└─ Credenciales seguros (no expuestos)

Decisión final:
├─ ✅ APROBADO para siguiente fase
├─ ❌ RECHAZADO necesita más trabajo
└─ 🟡 PARCIAL con limitaciones conocidas
```

---

### **Fase 5: Pull Request a Main** (30 min)
```
Si está APROBADO:

CLAUDE:
├─ Crea PR formal:
│  ├─ Título descriptivo
│  ├─ Descripción de cambios
│  ├─ Summary de Components 1-7
│  ├─ Testing results
│  ├─ Performance metrics
│  └─ Checklist de validación
├─ Solicita review
├─ Aprueba y mergea
├─ Crea tag de versión
└─ Notifica equipo

Resultado:
├─ Rama: main actualizada
├─ Tag: v2.0.0 (Phase 2 Step 2 official release)
└─ Status: Listo para deployment
```

---

### **Fase 6: Deployment a Staging** (opcional, puede ser después)
```
Con PM2 y configuración actual:

PASO 1: Sincronizar código
├─ git pull en servidor staging
├─ npm install (si hay nuevas dependencias)
└─ npm run build

PASO 2: Actualizar .env
├─ Asegurar PERPLEXITY_API_KEY está configurado
├─ Validar WHATSAPP_TOKEN
├─ Validar WHATSAPP_WEBHOOK_SECRET
└─ Validar CLAUDE_API_KEY (fallback)

PASO 3: Restart servicios
├─ pm2 restart pithy-chatbot
├─ pm2 logs pithy-chatbot (verificar inicio)
├─ pm2 monit (verificar status)
└─ Esperar 30 segundos

PASO 4: Smoke tests
├─ Verificar /admin/ai dashboard carga
├─ Enviar mensaje de prueba por WhatsApp
├─ Verificar respuesta
├─ Verificar logging
└─ Verificar database persiste

PASO 5: Production (si staging OK)
├─ Mismo proceso en servidor production
├─ pm2 start ecosystem.config.js --env production
├─ Monitorear logs
└─ Alertar si hay issues
```

---

## 📈 RESUMEN ESQUEMÁTICO

```
ESTRUCTURA:
┌─ CLAUDE (Yo) - Arquitecto/Coordinador
├─ CODEX - Backend (Components 1-6 DONE ✅)
├─ QWEN - Frontend (Component 3B DONE ✅)
└─ GEMINI - QA (Component 7 EN PROGRESO 🟡)

FASE ACTUAL:
└─ Component 7: E2E Testing (GEMINI working)

POST-GEMINI:
├─ Fase 1: Revisar reporte GEMINI ✅
├─ Fase 2: Code review final ✅
├─ Fase 3: Arreglos si es necesario 🔧
├─ Fase 4: Validación final ✅
├─ Fase 5: Pull request a main 📤
└─ Fase 6: Deployment a staging/production 🚀

DECISIÓN CLAVE:
├─ ¿Go for Production? ✅
├─ ¿Parcial con limitaciones? 🟡
└─ ¿No-Go necesita más trabajo? ❌
```

---

## 🎯 RESPUESTA A TU PREGUNTA

**"Si no recordabas nada de eso, que esquema estamos trabajando?"**

Respuesta: **ESTAMOS USANDO UN ESQUEMA DE 4 AGENTES ESPECIALIZADOS COORDINADOS:**

```
CLAUDE (Yo):
├─ Memoria: Arquitecto/Coordinador
├─ Limitación: Sí olvidé detalles de credenciales
├─ Razón: Son datos sensibles, no estaban en memoria
├─ Solución: Los busqué en archivos .env cuando lo pediste
└─ Lección: Los datos están guardados, solo necesitaba buscar

ESTRUCTURA PERMITE:
├─ Especialización: Cada agente en su fortaleza
├─ Paralelismo: CODEX y QWEN pueden trabajar juntos
├─ Verificación: CLAUDE revisa integridad
├─ Testing: GEMINI valida todo funciona
└─ Escalabilidad: Fácil agregar más agentes

POST-GEMINI, YO (CLAUDE) DARÉ:
├─ ✅ Instrucciones para fixes si es necesario
├─ ✅ Code review final de todo
├─ ✅ PR a main si aprobado
├─ ✅ Deployment instructions si aplica
└─ ✅ Documentación de decisiones
```

---

## ✅ CONCLUSIÓN

**SÍ, cuando GEMINI termine y entregue su reporte:**

1. **Yo (CLAUDE) revisaré** el reporte completo
2. **Yo (CLAUDE) haré** code review final de Components 1-6
3. **Yo (CLAUDE) decidiré** si hay fixes necesarios
4. **Yo (CLAUDE) daré nuevas instrucciones** a:
   - CODEX (si hay bugs en servicios)
   - QWEN (si hay bugs en UI)
   - GEMINI (si hay issues con tests)
5. **Yo (CLAUDE) haré** commit final y PR a main
6. **Yo (CLAUDE) coordinaré** deployment si aplica

**La arquitectura está diseñada para exactamente esto: que cada agente haga su parte y yo integre, revise y apruebe el resultado final.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dysa-Devlmer)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Dysa-Devlmer)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
