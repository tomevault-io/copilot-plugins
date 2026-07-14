## argentum

> Estamos construyendo un **Argentum Online nuevo**, con arquitectura moderna, desde cero. No es un refactor del codigo legacy — es una reimplantacion limpia que respeta las mecanicas del juego original pero implementa todo con patrones solidos, codigo testeable y features incrementales.

# Argentum Online - Reimplementacion desde Cero

## Que estamos haciendo

Estamos construyendo un **Argentum Online nuevo**, con arquitectura moderna, desde cero. No es un refactor del codigo legacy — es una reimplantacion limpia que respeta las mecanicas del juego original pero implementa todo con patrones solidos, codigo testeable y features incrementales.

El objetivo: un servidor y cliente funcionales, sin spaghetti de IFs, sin deuda tecnica heredada, y con la posibilidad de evolucionar sin miedo.

## Filosofia de desarrollo

- **Features incrementales y testeables**: cada feature se implementa completa (server + cliente + tests) antes de pasar a la siguiente. En cualquier momento del proyecto, el juego deberia poder compilar y ejecutarse.
- **Primero entender, despues codear**: la documentacion del juego (reglas, formulas, mecanicas) esta en los docs del server legacy. Leer antes de implementar.
- **Arquitectura sobre velocidad**: preferir codigo claro y mantenible sobre codigo rapido de escribir. Usar Specification Pattern, Event Bus, y Rules Engine donde corresponda para evitar el infierno de IFs anidados del server original.

## Estructura del proyecto

```
.
├── apps/
│   ├── client/          # Cliente nuevo (React/PixiJS/Tauri — TypeScript)
│   └── server/          # Servidor nuevo (a definir)
├── context/
│   └── reimplementacion/ # Planificacion y decisiones arquitectonicas
│       ├── arquitectura-patrones.md   # Patrones recomendados (Specs, Events, Rules)
│       └── consideraciones.md         # Decisiones pendientes y preguntas abiertas
├── legacy/
│   ├── legacy-server/   # Servidor original (documentacion de referencia)
│   │   └── docs/wiki/   # TODA la documentacion del juego: mecanicas, combate, magia, NPCs, etc.
│   ├── legacy-client-vb6/  # Cliente original en VB6 — completo y funcional
│   └── legacy-client-java/ # Cliente Java (referencia principal para la reimplementacion)
│                          # Mas moderno y mejor estructurado que el VB6, aunque incompleto
```

## Referencias legacy — como usarlas

### `legacy/legacy-server/docs/wiki/` — Fuente de verdad de las mecanicas

Aca esta TODA la documentacion del juego. Antes de implementar cualquier feature, leer el doc correspondiente:

| Feature | Documento |
|---|---|
| Arquitectura general del server | `01-ARQUITECTURA-GENERAL.md` |
| Personaje (stats, niveles, razas, clases) | `02-PERSONAJE.md` |
| Combate (ataque, defensa, PvP, PK) | `03-COMBATE.md` |
| Magia (hechizos, efectos) | `04-MAGIA.md` |
| NPCs e IA | `05-NPCs-E-IA.md` |
| Inventario y objetos | `06-INVENTARIO-Y-OBJETOS.md` |
| Oficios (pesca, mineria, etc.) | `07-OFICIOS-Y-TRABAJO.md` |
| Comercio y economia | `08-COMERCIO-Y-ECONOMIA.md` |
| Clanes | `09-CLANES.md` |
| Party | `10-PARTY.md` |
| Facciones y Pretorianos | `11-FACCIONES-Y-PRETORIANOS.md` |
| Mundo y mapas | `12-MUNDO-Y-MAPAS.md` |
| Protocolo y red | `13-PROTOCOLO-Y-RED.md` |
| Admin y seguridad | `14-ADMIN-Y-SEGURIDAD.md` |
| Consideraciones reimplantacion | `90-CONSIDERACIONES-REIMPLEMENTACION.md` |

### `legacy/legacy-client-java/` — Referencia principal del cliente

Este es el cliente que usamos como **referencia principal** para la mayoria de cosas. Fue un intento mas moderno y mejor pensado que el VB6. La estructura de paquetes, la separacion de responsabilidades y las decisiones de diseno aca son un buen punto de partida. Ojo: esta incompleto y tiene interfaces vacias en muchos lados.

### `legacy/legacy-client-vb6/` — Cliente original completo

El cliente VB6 original, completo y funcional contra el server legacy. Se referencia para entender el comportamiento exacto del juego (flujos de UI, validaciones del cliente, manejo de paquetes). No como modelo arquitectonico, sino como documentacion viva del comportamiento esperado.

## Decisiones arquitectonicas clave

Las decisiones y consideraciones estan documentadas en `context/reimplementacion/`. Aqui vamos a ir plasmando ideas y las vamos a ir trabajando con el agente de IA

## Al trabajar en el proyecto

1. **Leer la documentacion** del mecanismo que vas a implementar en `legacy/legacy-server/docs/wiki/`
2. **Revisar el cliente Java** (`legacy/legacy-client-java/`) como referencia de implementacion
3. **Consultar `context/reimplementacion/`** para ver decisiones arquitectonicas y consideraciones
4. **Implementar feature por feature**, con tests, manteniendo el juego siempre funcional
5. **Documentar decisiones nuevas** en `context/reimplementacion/` cuando se tome una decision arquitectonica relevante

## Stack del cliente nuevo

El cliente nuevo (`apps/client-pixi/`) ya tiene stack definido: TypeScript, React 19, PixiJS 8, Redux Toolkit, Vite, Biome, Tauri v2. Ver `apps/client-pixi/AGENTS.md` para detalles de build, test y convenciones del cliente.

El servidor (`apps/server/`) esta pendiente de definir stack.

---
> Source: [francollamas/argentum](https://github.com/francollamas/argentum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
