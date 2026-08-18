## close-task-release

> Al cerrar una tarea del backlog — commit, decisión de versión, asentar en la card, y preguntar si pushear


# Cierre de tarea → commit + versión + push?

Cuando una tarea del backlog (`.backlog/items/PD-*.md`) pasa a `done` (o se archiva un change OpenSpec asociado), **no cerrar en silencio**. Seguir este orden:

## 1. Commit antes de marcar `done`

- Si hay cambios sin commitear del trabajo de la tarea → crear el/los commits (solo si el usuario lo pide o confirma; si el cierre implica asentar, **proponer el commit** y ejecutarlo al confirmar).
- No mezclar work-in-progress de otras tareas. El commit debe corresponder al cierre.
- Mensaje: Conventional Commits (`feat:`, `fix:`, `chore:`, etc.).

## 2. Decidir versión (bump o no)

Leer `package.json` `version` actual. Proponer **una** opción SemVer y pedir confirmación:

| Decisión | Cuándo | Cómo |
|----------|--------|------|
| `none` | Docs, backlog-only, chore interno sin release | No correr scripts de versión |
| `patch` | Fix / mejora chica compatible | `npm run version:patch` |
| `minor` | Feature user-facing (ej. FEAT-*) | `npm run version:minor` |
| `major` | Breaking change | `npm run version:major` |

Si hay bump: actualizar `CHANGELOG.md` (Keep a Changelog) **antes o junto** al bump; los scripts crean commit `chore(release): v%s` + tag anotado.

## 3. Asentar en la tarea

En el body del item `done`, dejar explícito:

```markdown
**Release:** vX.Y.Z (patch|minor|major) — o `none` (sin bump; quedó en vX.Y.Z)
```

Bump `updated` al cerrar. Si hubo OpenSpec archive, linkear la carpeta archive.

## 4. Preguntar push

Después del commit (y del release commit/tag si hubo bump), **preguntar siempre**:

> ¿Pusheo a origin? (si hubo bump: `git push --follow-tags`)

No pushear hasta confirmación explícita. Nunca `--force` a `main`/`master`.

## Ejemplo (FEAT-06)

1. Commit del feature + artefactos SDD  
2. Propuesta: `minor` 0.2.3 → 0.3.0 (feature user-facing)  
3. Card: `**Release:** v0.3.0 (minor)`  
4. Preguntar push + `--follow-tags`

---
> Source: [rodrigoperaltadev/postdata-fun](https://github.com/rodrigoperaltadev/postdata-fun) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
