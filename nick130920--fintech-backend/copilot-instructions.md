## conventional-commits

> Enforce Conventional Commits and release note trailers


# Conventional Commits Rule

Cuando el usuario pida crear un commit, usa formato Conventional Commits:

`type(scope): resumen en imperativo`

Tipos permitidos:
- `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `chore`, `build`, `ci`, `revert`

Reglas:
- Mensaje corto en minúsculas (excepto nombres propios).
- Máximo 72 caracteres en la primera línea.
- Explicar el "por qué" en el cuerpo si hace falta.
- No usar mensajes genéricos como "update", "changes", "wip".

Para release notes automáticas en Codemagic, agregar trailers cuando aplique:
- `RN-es: texto de notas para Play Console en español`
- `RN-en: text for Play Console release notes in English`

Ejemplo válido:

`feat(reports): agrega exportación csv y pdf`

`RN-es: Exportación de transacciones en CSV y PDF desde Reportes.`
`RN-en: Added CSV and PDF transaction export from Reports.`

---
> Source: [nick130920/fintech-backend](https://github.com/nick130920/fintech-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
