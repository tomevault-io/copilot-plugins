## xthegospel

> Estructura de carpetas y mantenibilidad del código FamilyDash


# Architecture + Maintainability

## Estructura esperada

```text
src/
  components/  # Reutilizables puros, sin lógica de negocio
  features/    # Features completas (Tasks, Finances, Calendar, etc.)
  hooks/       # Custom hooks; nombres con prefijo use
  lib/         # Firebase config, utils, constants
  types/       # Interfaces y type aliases
  pages/       # Rutas; solo routing + layout
```

## Reglas de mantenibilidad

- Evitar archivos de más de 300 líneas; dividir en subcomponentes/módulos.
- Co-ubicar tests junto al componente (`X.tsx` + `X.test.tsx`).
- Evitar imports relativos con más de 2 niveles (`../../..`); usar alias `@/`.
- Evitar funciones de más de 50 líneas sin justificación.
- Evitar comentarios que describen lo obvio; preferir código claro.
- Evitar `TODO` sin issue de GitHub asociado.
- Evitar `!important` salvo caso documentado.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Victordaz07)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Victordaz07)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
