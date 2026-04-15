## arq-manager

> Antes de generar o modificar código, consulta los archivos de skill según la tarea:

# arq-manager — Reglas para Windsurf (Cascade)

## Skills del proyecto

Antes de generar o modificar código, consulta los archivos de skill según la tarea:

- Componentes React o Next.js → lee `.agents/skills/react-best-practices/SKILL.md`
- Patrones de composición, compound components, evitar prop proliferation → lee `.agents/skills/composition-patterns/SKILL.md`
- Revisión de UI, accesibilidad, diseño web → lee `.agents/skills/web-design-guidelines/SKILL.md`

## Reglas de código

- No crear props booleanas (isX, hasX) para variar comportamiento de componentes. Usa compound components o composición con `children`.
- Seguir las guías de performance de `react-best-practices`: lazy loading, memoización correcta, evitar re-renders innecesarios.
- Al revisar archivos de UI, aplicar los 100+ criterios de `web-design-guidelines` (accesibilidad, performance visual, UX).
- TypeScript estricto en todo el proyecto.
- Preferir `children` sobre render props para composición de componentes.

## Cuándo activar cada skill

Si el usuario menciona: "crea un componente", "refactoriza", "optimiza", "revisa performance" → usa `react-best-practices`
Si el usuario menciona: "compound component", "demasiadas props", "variantes", "reutilizable" → usa `composition-patterns`
Si el usuario menciona: "revisa el diseño", "accesibilidad", "UI review", "interfaz" → usa `web-design-guidelines`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/symbiosecorp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
