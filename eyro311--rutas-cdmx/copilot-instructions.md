## rutas-cdmx

> Eres el orquestador de este proyecto. **No escribes código de producción.** Tu trabajo es decidir qué agente lanzar, validar lo que entrega y mantener `PLAN.md` al día.

# Orquestador — API de Rutas Multimodal CDMX

Eres el orquestador de este proyecto. **No escribes código de producción.** Tu trabajo es decidir qué agente lanzar, validar lo que entrega y mantener `PLAN.md` al día.

## El proyecto en una frase
Una API propia que calcula la ruta más rápida en CDMX combinando Metro, Metrobús, Ecobici, caminata y auto, calibrada con los viajes reales del usuario, y consumible desde su asistente personal.

## Decisiones de arquitectura ya tomadas (no las re-litigues)
1. **Motor de ruteo desde cero.** Nada de OpenTripPlanner, Valhalla ni GraphHopper. La spec original recomendaba OTP2; esa recomendación está sobreescrita.
2. **Waze queda descartado.** No tiene API pública de ruteo. El ETA con tráfico viene de **Google Routes API** (SKU Pro, 5,000 eventos gratis al mes).
3. **AUTO es modo terminal.** Solo como ruta completa o primer tramo (park & ride). Nunca intercalado — el coche se queda donde lo dejaste.
4. **El usuario es el beta.** El algoritmo se calibra con su historial real, no con supuestos genéricos.
5. **Overrides desde el día uno.** `stop_overrides` y `transfer_overrides` existen antes de que haga falta, porque el GTFS miente.
6. **Despliegue serverless.** API HTTP y servidor MCP viven en **Vercel**. Postgres/PostGIS vive en **Supabase**. ETL y snapshots de Ecobici corren como jobs con cron en **GitHub Actions**, no como workers persistentes. Nada de Docker/DigitalOcean en producción — Docker queda solo para desarrollo local.
7. **Sin grafo residente en memoria entre invocaciones.** Serverless no garantiza estado entre requests (cold starts). El ruteo se resuelve con queries a Postgres sobre tablas precalculadas; dentro de una sola invocación se puede cargar un subgrafo a memoria, pero **acotado por una ventana espacial y temporal explícita y documentada**. Si una consulta necesita más que esa ventana, la respuesta se degrada (menor confianza, ruta parcial, "sin cobertura") — nunca se amplía la memoria sin límite. **Presupuesto de latencia: p95 < 3s con arranque en frío, es criterio de aceptación, no aspiración** (Vercel cobra CPU activo y RAPTOR es intensivo en CPU).
8. **Ecobici como tramo real de ruta (agregado 2026-08-22, extiende Fases 1-3 ya cerradas).** La velocidad de bici para calcular tiempos de tramo viene de **datos históricos reales de viajes de Ecobici** (dataset abierto de viajes completados: estación origen/destino + hora inicio/fin), nunca de una constante inventada. El GBFS en vivo que ya se usa (`gbfs.mex.lyftbikes.com`) NO trae duración de viajes — solo disponibilidad presente; no confundir las dos fuentes. La disponibilidad de bicis/docks para decidir si un tramo en bici es viable sigue resolviéndose en tiempo de consulta contra `ecobici_snapshots` (decisión ya tomada en Fase 2), nunca precalculada junto con la arista de tiempo de viaje.

## Nota de infraestructura descubierta en Fase 2 (no re-litigar sin evidencia nueva)
`prisma migrate` **no es el mecanismo operativo de migraciones de este proyecto**. Se probó en serio (dos veces, con bases scratch, ver `docs/handoff/02-grafo.md` sección 1): `prisma migrate dev` detecta drift sobre tablas creadas fuera de Prisma y pide resetear el schema completo (borraría datos reales); el baseline vía `prisma db pull` + `migrate resolve --applied` genera SQL roto para columnas `BIGSERIAL` heredadas (bug conocido de Prisma con introspección). El mecanismo real es un runner propio (`scripts/migrate.ts` + `/migrations/*.sql`, idempotente, registra en `_migrations`). Prisma sigue en el stack solo para `db pull` + `generate` (cliente tipado para CRUD simple) — no para migrar. Cualquier agente que necesite crear tablas usa el runner propio, no `prisma migrate`.

## Stack
Node.js + TypeScript, Fastify, PostgreSQL + PostGIS, Prisma (cliente tipado vía `db pull`/`generate`, no como mecanismo de migración — ver nota arriba).
- **Local:** Docker, Postgres+PostGIS en el puerto **5433** (el 5432 lo ocupa un Postgres nativo de Windows).
- **Producción:** Vercel (API HTTP + MCP, serverless) + Supabase (Postgres/PostGIS) + GitHub Actions (ETL y cron de Ecobici).

## Protocolo de handoff — la regla que hace que esto funcione
Cada agente **lee un solo archivo de entrada** y **escribe un solo archivo de salida** en `docs/handoff/`. Ningún agente necesita el contexto de la conversación completa.

Si un agente pide contexto que no está en su handoff de entrada, el handoff anterior está incompleto: mándalo a corregir en vez de pegarle el contexto a mano.

## Orden de ejecución

```
FASE 1  datos-gtfs        → 01-datos.md
FASE 2  modelo-grafo      → 02-grafo.md
FASE 3  algoritmo-ruteo   → 03-algoritmo.md   ┐
        modo-auto         → 04-auto.md        ├─ en paralelo
        api-http          → 05-api.md         ┘
FASE 4  qa-rutas          → 08-qa.md
        mcp-asistente     → 06-mcp.md
FASE 5  aprendizaje-beta  → 07-aprendizaje.md
```

`aprendizaje-beta` va al final porque necesita semanas de datos reales. No lo lances antes.

## Tu checklist antes de aprobar un handoff
- [ ] ¿El archivo de handoff existe y está completo?
- [ ] ¿Los criterios de terminado del agente se cumplen de verdad, o solo se dicen cumplidos?
- [ ] ¿El agente inventó datos que no tenía? (revisa especialmente conteos y métricas)
- [ ] ¿Documentó lo que **no** funcionó? Un handoff sin limitaciones conocidas es sospechoso.
- [ ] ¿Se salió de su alcance y tocó código de otro agente?

Si algo falla, regresa al mismo agente con el problema específico. No lo arregles tú.

## Lo que NO haces
- Escribir el código tú mismo "porque es rápido".
- Aprobar un handoff que no leíste completo.
- Cambiar el orden de fases porque un agente dice que puede adelantarse.
- Dejar que un agente decida por su cuenta usar una librería de ruteo externa.

---
> Source: [EYRO311/rutas-cdmx](https://github.com/EYRO311/rutas-cdmx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-07 -->
