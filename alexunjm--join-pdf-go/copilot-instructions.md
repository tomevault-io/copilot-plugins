## join-pdf-go

> Estándares de código — nomenclatura, estructura de imports, manejo de errores


# Estándares de código

Consultar y seguir `docs/architecture/coding-standards.md` cuando exista en el proyecto. Resumen de obligatorios:

## Nomenclatura

- **Variables y funciones/métodos:** camelCase.
- **Clases, interfaces y tipos:** PascalCase, con sufijos descriptivos (Component, Service, Repository, etc.) según el stack.
- **Constantes:** SCREAMING_SNAKE_CASE (cuando el lenguaje lo permita).
- **Archivos:** según convención del stack (p. ej. kebab-case para Angular, PascalCase para Java).

## Estructura de código

- **Imports:** orden definido en `coding-standards.md` (p. ej. stdlib → librerías externas → internos → relativos), agrupados y ordenados alfabéticamente dentro de cada grupo.
- **Estructura de clases/componentes:** dependencias inyectadas, constructor, métodos públicos; secciones claras sin mezclar responsabilidades.

## Manejo de errores

- Usar excepciones o errores **específicos del dominio**, con mensajes claros y contextualizados.
- Evitar excepciones genéricas sin contexto (ej. `throw new Error("Error")` o `RuntimeException("Error")`).
- En servicios asíncronos/reactivos, manejar errores en el flujo (p. ej. operadores de error en RxJS) y propagar de forma controlada.

## Consistencia

- Mantener el mismo estilo que el archivo y el proyecto (indentación, comillas, punto y coma si aplica).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alexunjm)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alexunjm)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
