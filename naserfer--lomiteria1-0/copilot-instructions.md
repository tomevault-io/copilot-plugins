## no-git-without-permission

> Nunca ejecutar git push, commit o git add . sin permiso explícito del usuario


# Git: solo con permiso

- **No ejecutar** `git add .`, `git add -A` ni ningún `git add` que afecte todo el repo sin que el usuario lo pida o lo autorice.
- **No hacer** `git commit` ni sugerir commits automáticos; solo si el usuario lo pide explícitamente.
- **No hacer** `git push` ni sugerir push sin que el usuario lo autorice.

Si hace falta versionar cambios, indicar qué comandos ejecutar y dejar que el usuario decida cuándo y cómo (por ejemplo: "Cuando quieras, puedes hacer `git add …` y `git commit`").

---
> Source: [naserfer/Lomiteria1.0](https://github.com/naserfer/Lomiteria1.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
