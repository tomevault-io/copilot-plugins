## memoria-historica

> Aplicación web pública y sin ánimo de lucro para ayudar a familiares de personas desaparecidas durante la Guerra Civil española y la posguerra a averiguar qué ocurrió con sus seres queridos.

# Memoria Histórica — Contexto del proyecto

Aplicación web pública y sin ánimo de lucro para ayudar a familiares de personas desaparecidas durante la Guerra Civil española y la posguerra a averiguar qué ocurrió con sus seres queridos.

**Estado actual:** fase de definición y planificación. Aún no se ha tocado código. La fase de entrevista de requisitos está cerrada y aprobada por el usuario el 2026-06-09.

## Documentos clave

- [docs/vision.md](docs/vision.md) — visión completa, alcance, modelo de datos, criterios de calidad.
- [docs/decisiones.md](docs/decisiones.md) — decisiones tomadas con su justificación (para no rediscutirlas).
- [docs/roadmap.md](docs/roadmap.md) — siguientes pasos antes de empezar a construir.
- [docs/auditoria-fuentes.md](docs/auditoria-fuentes.md) — auditoría completa (1ª, 2ª, 3ª pasada y validación real, 2026-06-09).
- [docs/plan-tecnico.md](docs/plan-tecnico.md) — borrador del plan técnico: stack, modelo de datos, wireframes, matching, ingesta.
- [docs/caso-piloto.md](docs/caso-piloto.md) — Luciano Herrera Calonge, caso real con el que validar el MVP.

## Reglas para futuras sesiones

- **No agotar los build minutes de Netlify (free tier).** Cada push a `main` dispara un build remoto. Validar siempre en local antes (`pnpm typecheck`, `pnpm build`, `pnpm dev`), agrupar varios commits en un solo push significativo, y usar `[skip ci]` en el commit message cuando el cambio sea solo de `docs/`, `README.md` o cosas que no necesitan redeploy. No hacer commits vacíos para "disparar deploy" salvo necesidad real.
- **Mantener la documentación viva.** Siempre que se tome una decisión nueva, se cierre una pendiente, se avance en el roadmap, se descarte una hipótesis o cambie el alcance, actualizar el documento correspondiente en el mismo turno. La documentación debe reflejar el estado actual del proyecto en todo momento, no quedarse desfasada.
- **No empezar a construir código** hasta completar los pasos pendientes del roadmap (auditoría de fuentes + plan técnico + wireframes).
- Antes de proponer cambios al alcance, revisar `decisiones.md` — muchas decisiones ya se tomaron con justificación; no reabrir sin motivo nuevo.
- Tono y lenguaje del producto: claro, sin jerga archivística, accesible para familiares sin conocimientos previos.
- Sensibilidad del dominio: se trata de víctimas de represión política. Tratar el contenido con respeto y cuidado ético.
- El proyecto es sin ánimo de lucro y web-only (responsive). No proponer monetización ni app nativa salvo que el usuario lo pida.

---
> Source: [astrozeta/memoria-historica](https://github.com/astrozeta/memoria-historica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
