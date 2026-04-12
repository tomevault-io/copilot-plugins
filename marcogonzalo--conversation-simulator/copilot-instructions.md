## conversation-simulator

> Reglas específicas para el frontend Next.js/React


# Frontend - Next.js/React

## Diseño y UI

- Siempre implementa modo claro y oscuro, modo oscuro como predeterminado
- Ejecuta build después de cambios importantes en frontend y corrige errores hasta que pasen
- Mantén componentes pequeños y reutilizables con props bien tipadas
- Diseña mobile-first con breakpoints de Tailwind

## Estado y Gestión

- Usa Zustand para estado global, useState para estado local
- Implementa lazy loading para componentes y rutas
- Mantén bundles de frontend optimizados

## TypeScript

- Usa tipado estricto, evita 'any' y 'unknown' cuando sea posible
- Usa imports absolutos cuando sea posible
- Evita imports relativos complejos

## Performance

- Analiza y optimiza el tamaño de bundles regularmente
- Implementa cache inteligente para configuraciones de voz y personas
- Mantén hot reload funcionando para desarrollo eficiente

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcogonzalo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/marcogonzalo)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
