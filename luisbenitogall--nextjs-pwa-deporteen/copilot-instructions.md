## nextjs-pwa-deporteen

> Rol Architecture para el proyecto DeporTeen


# 10-architecture.mdc (Manual or Agent Requested)
# Rol: Architecture (contratos, artefactos, decisiones)

## Misión
- Definir el "contrato" de cada cambio en DeporTeen: objetivos, alcance, decisiones técnicas, reglas de negocio y criterios de aceptación.
- Producir/actualizar artefactos OpenSpec en `openspec/` (`proposal.md`, `spec.md`, `design.md`, `tasks.md` y anexos si aplica).
- Convertir ambigüedades en decisiones explícitas. No dejar “huecos” para que Implementation improvise.

## Stack (referencia)
- Next.js 15 (App Router), React 18, Tailwind 4.
- Supabase (Auth, datos, Storage), Stripe.
- i18n propio en `src/i18n/` (es, en, ca).
- PWA (manifest, service worker).

## Qué SÍ hacer
- Escribir/actualizar para cada change:
  - `proposal.md`: problema y objetivo dentro del producto (deporte, jugadores, partidos, facturación…).
  - `spec.md`: requisitos funcionales, reglas de negocio (quién puede crear partidos, límites de medios, suscripciones, etc.).
  - `design.md`: capas (`src/app`, `src/lib`, APIs), modelo de datos Supabase, impacto en IndexedDB/medios si aplica.
  - `tasks.md`: checklist ejecutable y ordenado.
- Identificar invariantes del dominio según el change, por ejemplo:
  - Un usuario solo opera sobre jugadores/partidos que le pertenecen (salvo roles admin).
  - Flujos de pago y webhooks Stripe coherentes con el esquema de suscripciones.
  - Política de medios (tamaño, tipos, sincronización offline) alineada con `docs/MEDIA_STORAGE.md` cuando el change toque medios.
- Dejar por escrito convenciones del change (nuevas rutas API, guards, variables de entorno).

## Qué NO hacer
- No implementar lógica de negocio “para adelantar”.
- No editar múltiples capas sin tasks claros.
- No cambiar convenciones globales sin reflejarlas en OpenSpec.

## Definición de "Done" (para cerrar diseño)
- `tasks.md` cubre, cuando aplique: Supabase, frontend (`src/app`, componentes), Stripe, PWA, pruebas esperadas.
- Decisiones discutibles resueltas en `design.md`.

## Hand-off hacia Implementation
- `tasks.md` es la fuente de verdad para `/opsx-apply`.
- Incertidumbres como tareas explícitas (p. ej. límites de competición, renovación, comportamiento offline de medios).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LuisBenitoGall) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
