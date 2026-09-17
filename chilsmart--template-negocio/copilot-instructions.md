## template-negocio

> > Antes de llenar esto, mira `CLAUDE.ejemplo.md` — el contexto del proyecto que se ve en el video

# CLAUDE.md — Contexto del proyecto

> Antes de llenar esto, mira `CLAUDE.ejemplo.md` — el contexto del proyecto que se ve en el video
> (un sistema de compra-venta). El tuyo no tiene que parecerse en nada a ese ejemplo.

## Qué es este proyecto

<Una o dos frases: qué es, para quién.>

**El problema:** <qué hace hoy la gente sin esto — a mano, en planillas, perdiendo tiempo, etc.>

**La solución:** <qué va a hacer tu proyecto, en 2-4 puntos.>

## Stack

<Qué vas a usar para construirlo. No hace falta Next.js/Turso/Vercel si tu proyecto no es una app
web — ver la nota más abajo.>

## Las 6 fases de SDD que seguimos

1. **Planificación** — Problema identificado (arriba)
2. **Análisis** — Ver `SPEC.md` (qué debe hacer exactamente)
3. **Diseño** — Stack definido (arriba) + arquitectura en `agents.md`
4. **Implementación** — Código (tú lo haces, con tu agente)
5. **Testing** — Ver criterio de éxito en `SPEC.md`
6. **Despliegue** — Solo si tu proyecto es una app que se publica

## Cómo tu agente te puede ayudar

### En esta carpeta, tu agente ve:

- `CLAUDE.md` ← contexto (esto)
- `SPEC.md` ← qué debe hacer (sin código)
- `agents.md` ← agentes + MCPs + Skills que usa tu proyecto
- `conditions.md` ← reglas del sistema

### Cuando pides ayuda:

- Si algo cambió, actualiza `SPEC.md` o `conditions.md` primero
- Después pide la implementación — tu agente ya tiene el contexto

## Tu vault — que tu proyecto tenga memoria

Crea una carpeta `<tu-proyecto>-vault/` junto a este repo. La skill que la alimenta y la regla
canario vienen del kit base
([`chilsmart/config-agente`](https://github.com/chilsmart/config-agente)) — cópialo a este
proyecto antes de empezar, no lo dupliques acá. Regla de oro: si no está en el vault, no pasó.

## Nota sobre el stack

Si tu proyecto no es una app web (por ejemplo, automatizar algo que hoy haces a mano, o un
análisis puntual), no necesitas Next.js/Turso/Vercel — usa lo que tenga sentido para lo que estás
construyendo. El método (spec → agentes → condiciones → construir) aplica igual.

## Cómo trabajar

1. Cambia algo en `SPEC.md` o `conditions.md`
2. Pídele a tu agente que lo implemente
3. Tu agente lee el contexto → entiende qué hacer
4. Commitea cuando termines un bloque, no micro-commits

## Referencias

- Clase: **Spec-Driven Development**
- Método: Planificación → Análisis → Diseño → Implementación → Testing → Despliegue

---

**Nota:** Este contexto ayuda a tu agente a entenderte mejor. Si haces cambios en `SPEC.md` o
`agents.md`, actualiza esto también.

---
> Source: [chilsmart/template-negocio](https://github.com/chilsmart/template-negocio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
