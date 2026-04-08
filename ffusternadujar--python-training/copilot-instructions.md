## python-training

> Este archivo documenta las instrucciones y configuraciones para agentes de IA que trabajan en este repositorio.

# AGENTS.md

Este archivo documenta las instrucciones y configuraciones para agentes de IA que trabajan en este repositorio.

## Comandos de Construcción y Pruebas

- Ejecutar scripts Python: `python script_name.py` o `python3 script_name.py`
- Ejecutar todas las pruebas: `pytest`
- Ejecutar un archivo de prueba específico: `pytest path/to/test_file.py`
- Ejecutar una prueba específica: `pytest path/to/test_file.py::test_function_name`
- Linting: `ruff check .` o `flake8 .`
- Formatear código: `ruff format .` o `black .`
- Verificación de tipos: `mypy .`

## Guías de Estilo de Código

### Python
- **Versión**: Python 3.x moderno
- **Importaciones**: Agrupar en orden: biblioteca estándar, terceros, locales. Usar importaciones absolutas
- **Formato**: Seguir PEP 8 (4 espacios de indentación, 79-88 caracteres por línea)
- **Type Hints**: Usar anotaciones de tipo en firmas de funciones
- **Nomenclatura**:
  - `snake_case` para funciones y variables
  - `PascalCase` para clases
  - `UPPER_CASE` para constantes
- **Manejo de Errores**: Usar tipos de excepción específicos, evitar `except` genéricos
- **Documentación**: Añadir docstrings a funciones y clases explicando propósito y parámetros
- **Comentarios**: Escribir comentarios claros y concisos con fines educativos

## Descripción General del Proyecto

Este es un proyecto privado de formación en Python enfocado en aprender los fundamentos del lenguaje. El repositorio está en sus etapas iniciales y contendrá scripts de Python, ejercicios y materiales de aprendizaje a medida que avance la formación. El objetivo es crear una colección organizada de ejemplos y ejercicios que demuestren conceptos clave de Python.

## Estructura del Proyecto

Organizar el código lógicamente por tema o capítulo:
- `basics/` - Fundamentos del lenguaje
- `data_structures/` - Estructuras de datos
- `oop/` - Programación orientada a objetos
- `functional/` - Programación funcional
- `modules/` - Módulos y paquetes
- `testing/` - Pruebas y testing

Mantener scripts de ejemplo autocontenidos y bien documentados para referencia.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ffusternadujar)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ffusternadujar)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
