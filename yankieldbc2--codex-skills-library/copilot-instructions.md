## codex-skills-library

> Este repositorio es una biblioteca de skills de Codex. Mantener cambios pequenos, validados y seguros.

# AGENTS.md

Este repositorio es una biblioteca de skills de Codex. Mantener cambios pequenos, validados y seguros.

## Proposito

Antes de editar, entiende si el cambio afecta a:

1. un skill en `.agents/skills/<skill-name>/SKILL.md`;
2. la especificacion o calidad de skills en `docs/`;
3. los generadores y validadores en `scripts/`;
4. el paquete estable en `plugins/skills-pack/`;
5. los artefactos generados en `generated/` o `CATALOG.md`.

## Skill routing by default

- Before any task, choose the most specific applicable skill or skills.
- If multiple skills fit, use the smallest set that fully covers the task.
- If no exact skill fits, fall back to `token-efficient-codex-run` and say so explicitly.
- For unknown or stale repos, include `project-recovery-audit`.
- For bootstrap or takeover, use `codex-project-intake`.
- Do not ask the user for examples when a matching skill already exists in this repo.

## Hard file-size rule

- Code-bearing files must stay at or below 400 lines.
- If a code file is approaching the limit, refactor it before it crosses 400 lines.
- If a file is already at or above 400 lines and needs refactoring, create a checkpoint first: copy the original file to a backup path or save a patch diff that can be compared or restored later.
- Keep the checkpoint until the refactor passes validation.

## Stack actual

- Node.js `>=20`
- ESM (`"type": "module"`)
- npm
- Tests con `node --test`
- CI en GitHub Actions
- Sin dependencias runtime declaradas en `package.json`

## Orden de lectura

Antes de cambiar algo, lee solo lo necesario:

1. `docs/CODEBASE_INDEX.md`
2. `docs/SKILL_SPEC.md`
3. `docs/QUALITY_RUBRIC.md`
4. `docs/SKILL_ROUTING.md` para la politica de selection de skills
5. `docs/PROJECT_INTAKE.md` para bootstrap y takeover
6. `CONTRIBUTING.md` cuando agregues o cambies skills
7. El `SKILL.md` afectado y sus `references/` si la tarea toca un skill concreto

## Reglas del repositorio

- `.agents/skills/` es la fuente de verdad para skills editables.
- `plugins/skills-pack/` es una copia sincronizada de los skills estables.
- `CATALOG.md` y `generated/` son artefactos generados; no se editan a mano.
- Los ejemplos largos van en `references/`, no dentro de `SKILL.md`.
- Los helpers deterministas van en `scripts/`.
- No agregues secretos, tokens, `.env`, claves privadas ni datos de cliente.
- Mantiene cada skill con un solo objetivo claro.
- Si cambias metadata de skills, mantenla alineada con `docs/SKILL_SPEC.md`.
- Si una regla se repite, conviertela en skill, plantilla o documento reutilizable.
- Si necesitas medir mejora real, usa `docs/EFFICIENCY_SYSTEM.md` y `templates/project/EFFICIENCY_RUN.json`.
- Si dudas entre skills, revisa `docs/SKILL_ROUTING.md` antes de improvisar.
- Usa `npm run check:loc` para validar el limite de 400 lineas en archivos de codigo.

## Reglas para skills

- Cada `SKILL.md` debe tener `Goal`, `Workflow` y `Validation`.
- La descripcion de frontmatter debe decir cuando usar el skill.
- El nombre del skill debe coincidir con la carpeta.
- El `SKILL.md` debe permanecer compacto; mueve detalles extensos a `references/`.
- Los skills estables son los unicos que se copian a `plugins/skills-pack/`.

## Reglas frontend y visuales

En este repo, "frontend" significa skills y documentos de UI/UX, no una app web runtime.

- Para skills de UI, usa referencias concretas en `references/` o `assets/`.
- Prioriza criterios de responsive, accesibilidad, jerarquia visual y QA final.
- Evita consejos genericos; escribe instrucciones accionables y comprobables.
- Si un skill toca diseno visual, documenta la intencion del asset y su origen en `docs/ASSET_MANIFEST.md`.
- Cuando haga sentido, apoya el trabajo con skills como `frontend-production-ui`, `premium-frontend-ui-design-v3`, `mobile-density-ui-system-v2`, `visual-polish-ai-degenericizer-v2` y `pixel-perfect-qa`.

## Flujo de assets

- Assets locales del skill: `.agents/skills/<skill-name>/assets/`
- Referencias largas: `.agents/skills/<skill-name>/references/`
- Helpers del skill: `.agents/skills/<skill-name>/scripts/`
- Paquete estable sincronizado: `plugins/skills-pack/`
- Salida generada: `generated/` y `CATALOG.md`
- Si agregas o regeneras assets relevantes, actualiza `docs/ASSET_MANIFEST.md`
- Nunca edites copias generadas de forma manual

## Comandos utiles

```bash
npm install
npm run validate
npm test
npm run catalog
npm run catalog:categories
npm run sync:plugin
npm run compare:efficiency -- <control.json> <treatment.json>
npm run check:duplicates
npm run check:secrets
npm run check:size
```

## Validacion requerida

Usa el conjunto mas pequeno que demuestre el cambio:

```bash
npm run validate
npm test
```

Si cambias metadata o reglas de generacion:

```bash
npm run catalog
npm run sync:plugin
```

Si tocas solo un skill, ademas confirma:

- `npm run check:duplicates` cuando cambies nombres o scopes similares
- `npm run check:secrets` cuando agregues ejemplos o referencias
- `npm run check:size` cuando un `SKILL.md` crezca

## Definition of Done

No cierres una tarea hasta verificar que:

- el cambio requerido esta implementado o documentado;
- no se editaron archivos generados a mano;
- la validacion relevante paso o se explico por que se omitio;
- el diff solo contiene cambios necesarios;
- reportaste archivos cambiados, validacion ejecutada, riesgos y siguiente paso.

---
> Source: [YankielDBC2/codex-skills-library](https://github.com/YankielDBC2/codex-skills-library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
