## rust-performance

> Este repositorio es parte de la colección complementaria de Jeresoft Academy y

# AGENTS.md

Este repositorio es parte de la colección complementaria de Jeresoft Academy y
se rige por RFC-0001 y RFC-0002.

## Objetivo

Construir un libro de ingeniería y crate educativo sobre rendimiento en Rust.
Toda afirmación de rendimiento necesita contexto, una hipótesis y evidencia
reproducible; una optimización ilegible o incorrecta no es una mejora.

## Antes De Escribir Código

Siempre, en este orden (RFC-0001 §2 y §13):

1. Explicar el concepto.
2. Explicar el problema medible.
3. Comparar alternativas.
4. Justificar la implementación y la medición.

## Código Y Medición

- Rust idiomático, `rustfmt` sin diffs y Clippy limpio.
- Sin `unsafe` ni dependencias externas no triviales sin aprobación humana.
- Toda medición declara entrada, entorno, línea base y limitaciones.
- No afirmar causalidad a partir de una sola ejecución ni optimizar antes de
  medir.
- Las pruebas verifican corrección; los benchmarks respaldan afirmaciones de
  rendimiento, pero no sustituyen las pruebas.

## Flujo De Entrega

- Crear el plan, el GitHub Project, milestones e issues antes de capítulos.
- Cada issue accionable se asigna a `jeresoftx`, tiene labels, milestone y
  pertenece al GitHub Project del curso.
- Cada PR resuelve un issue, conserva su milestone y labels, está asignado a
  `jeresoftx` y se agrega como ítem del mismo Project. La asociación
  issue–PR–Project debe verificarse antes de revisión o fusión.
- En revisión diferida se aplica RFC-0001 §20 y RFC-0002, sin marcar contenido
  como `reviewed` ni `published`.

## Documentación

Cada capítulo seguirá RFC-0001 §14 y §16. Incluirá modelo, diagramas Mermaid,
ejemplos progresivos, ejercicios, soluciones, pruebas y un benchmark útil o una
explicación honesta de por qué no aplica.

---
> Source: [jeresoftx/rust-performance](https://github.com/jeresoftx/rust-performance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
