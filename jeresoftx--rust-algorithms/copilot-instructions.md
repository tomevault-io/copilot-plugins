## rust-algorithms

> Este repositorio es parte de la colección núcleo técnico (algoritmos) de

# AGENTS.md

Este repositorio es parte de la colección núcleo técnico (algoritmos) de
Jeresoft Academy y se rige por la RFC-0001 (manual fundacional).

## Objetivo

Crear el mejor recurso educativo posible sobre algoritmos, estructuras de datos
y patrones de resolución de problemas en Rust.

Todo cambio debe mejorar simultáneamente:

- calidad técnica
- claridad
- documentación
- mantenibilidad

## Antes de escribir código

Siempre, en este orden (RFC-0001 §13):

1. Explicar el concepto.
2. Explicar el problema.
3. Comparar alternativas.
4. Justificar la implementación.

## Código

Conforme a RFC-0001 §13:

- Rust idiomático
- Clippy limpio y rustfmt sin diffs
- Sin `unsafe` salvo justificación documentada (comentario SAFETY)
- Comentarios donde aporten valor

## Documentación

Todo capítulo sigue las doce secciones de RFC-0001 §14 y la plantilla de §16.
Toda nueva funcionalidad incluye:

- README actualizado
- Diagramas Mermaid (RFC-0001 §12)
- Ejemplos ejecutables
- Tests
- Benchmarks (si aplica; si no aplica, se declara)

## Nunca

- Agregar dependencias innecesarias.
- Optimizar prematuramente.
- Duplicar código.
- Omitir documentación.
- Publicar capítulos parciales.

## Filosofía

Este repositorio debe poder utilizarse como un libro de ingeniería. Nunca
sacrificar claridad por ingenio. Explicar el porqué, no solo el cómo.

---
> Source: [jeresoftx/rust-algorithms](https://github.com/jeresoftx/rust-algorithms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
