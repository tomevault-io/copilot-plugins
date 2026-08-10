## rust-async

> Este repositorio es parte de la colección complementaria de Jeresoft Academy y

# AGENTS.md

Este repositorio es parte de la colección complementaria de Jeresoft Academy y
se rige por RFC-0001, el manual fundacional.

## Objetivo

Crear el mejor recurso educativo posible sobre programación asíncrona en Rust,
desde el protocolo `Future` hasta runtimes de producción y actores.

Todo cambio debe mejorar simultáneamente:

- calidad técnica;
- claridad;
- documentación;
- mantenibilidad.

## Antes De Escribir Código

Siempre, en este orden (RFC-0001 §2 y §13):

1. Explicar el concepto.
2. Explicar el problema.
3. Comparar alternativas.
4. Justificar la implementación.

## Código

- Rust idiomático, `rustfmt` sin diffs y Clippy limpio.
- Sin `unsafe`. Si un tema futuro necesitara estudiarlo, requiere decisión y
  justificación humana previa.
- Sin dependencias externas no triviales sin aprobación humana explícita.
- Las pruebas deben evitar depender de temporización real cuando un modelo
  determinista pueda expresar el invariante.
- Tokio se introduce después del executor educativo; no lo sustituye.

## Documentación

Todo capítulo sigue RFC-0001 §14 y §16. Incluye documentación, diagramas
Mermaid, ejemplos ejecutables, pruebas, ejercicios y benchmarks cuando apliquen
o una explicación de por qué no aplican.

## Flujo De Entrega

- El plan se convierte en milestones e issues antes de implementar capítulos.
- Cada issue accionable se asigna a `jeresoftx`, tiene labels, milestone y
  pertenece al GitHub Project del curso.
- Cada PR conserva la trazabilidad del issue y se valida antes de fusionarse.
- Cuando Joel autorice revisión diferida, se aplica RFC-0001 §20 sin marcar
  contenido como `reviewed` ni `published`.

## Nunca

- Agregar dependencias innecesarias.
- Optimizar antes de medir.
- Duplicar código.
- Omitir documentación.
- Presentar la asincronía como paralelismo automático.
- Publicar capítulos parciales.

---
> Source: [jeresoftx/rust-async](https://github.com/jeresoftx/rust-async) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
