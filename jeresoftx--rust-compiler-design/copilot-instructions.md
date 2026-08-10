## rust-compiler-design

> Este repositorio pertenece a la colección complementaria de Jeresoft Academy y

# AGENTS.md

Este repositorio pertenece a la colección complementaria de Jeresoft Academy y
se rige por RFC-0001 y RFC-0002.

## Objetivo

Construir un libro de ingeniería y crate educativo sobre diseño de compiladores
en Rust. El curso sigue el pipeline completo de un lenguaje pequeño: lexer,
parser, AST, análisis semántico, IR, optimizador, bytecode y máquina virtual.

## Antes De Escribir Código

Siempre, en este orden (RFC-0001 §2 y §13):

1. Explicar el concepto.
2. Explicar el problema que resuelve dentro del compilador.
3. Comparar alternativas de representación o ejecución.
4. Justificar la implementación mínima y sus invariantes.

## Límites Técnicos

- Rust idiomático, `rustfmt` sin diffs y Clippy limpio.
- Sin `unsafe`, nightly ni dependencias externas no triviales sin autorización
  humana explícita.
- El lenguaje educativo permanece pequeño; cada fase conserva diagnósticos y
  resultados verificables antes de optimizar o añadir sintaxis.
- Las pruebas comprueban tokens, árboles, diagnósticos, IR, bytecode y resultado
  observable de la máquina virtual.

## Flujo De Entrega

- Crear el plan, el GitHub Project, milestones e issues antes de implementar el
  compilador.
- Cada issue accionable se asigna a `jeresoftx`, tiene labels, milestone y
  pertenece al GitHub Project del curso.
- Cada PR resuelve un issue, conserva milestone y labels, está asignado a
  `jeresoftx` y se agrega al mismo Project. La asociación issue–PR–Project se
  verifica antes de revisión o fusión.
- En revisión diferida se aplica RFC-0001 §20 y RFC-0002, sin marcar capítulos
  como `reviewed` ni `published`.

## Documentación

Cada capítulo seguirá RFC-0001 §14 y §16. Incluirá modelo, diagrama Mermaid,
ejemplos progresivos, ejercicios, soluciones, pruebas y una declaración honesta
cuando complejidad o benchmark no apliquen.

---
> Source: [jeresoftx/rust-compiler-design](https://github.com/jeresoftx/rust-compiler-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
