## juan-bajana

> Estas reglas aplican a todo el repositorio y deben cumplirse antes, durante y después de cualquier modificación.

# Reglas obligatorias de trabajo

Estas reglas aplican a todo el repositorio y deben cumplirse antes, durante y después de cualquier modificación.

## Protección de ramas

- Nunca trabajar directamente sobre `main`.
- Crear una rama específica para cada tarea.
- Subir únicamente la rama de trabajo.
- Abrir un pull request hacia `main`.
- No fusionar el pull request automáticamente.

## Preparación antes de modificar código

- Antes de modificar código, ejecutar `git status`.
- Actualizar las referencias remotas antes de comenzar.
- Confirmar la rama actual y el estado del repositorio.
- Detenerse si existen cambios locales no relacionados con la tarea.

## Protección de información y datos

- No borrar datos, migraciones ni archivos sin verificar previamente su uso.
- No ejecutar migraciones destructivas sin autorización explícita.
- No modificar archivos `.env`.
- No publicar claves, contraseñas, tokens ni otros secretos.
- No sobrescribir información existente de usuarios, pacientes, animales, granjas, historiales o registros.

## Desarrollo y validación

- Limitar los cambios exclusivamente al alcance solicitado.
- Evitar refactorizaciones innecesarias.
- Ejecutar todas las pruebas disponibles relacionadas con los cambios.
- Revisar `git diff` antes de confirmar cambios.
- Verificar que no existan archivos o cambios accidentales.
- Detenerse si existen pruebas fallidas o cambios no relacionados.

## Commits y pull requests

- Crear commits claros, pequeños y relacionados con una sola tarea.
- No incluir cambios ajenos al objetivo del commit.
- Subir únicamente la rama de trabajo.
- Abrir un pull request hacia `main`.
- No fusionar el pull request automáticamente.
- Explicar en el pull request qué se modificó, por qué, qué pruebas se ejecutaron y qué riesgos o pendientes existen.

---
> Source: [elranchodejuan-jo/juan.bajana](https://github.com/elranchodejuan-jo/juan.bajana) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
