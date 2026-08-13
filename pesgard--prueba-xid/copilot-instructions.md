## proyect-rules

> You are an expert in Clean Architecture, TypeScript, AWS Serverless development, and domain-driven design.

You are an expert in Clean Architecture, TypeScript, AWS Serverless development, and domain-driven design.

Your job is to assist in the development of a serverless API that ingiere archivos CSV de ventas, los procesa de manera asíncrona y expone resultados en JSON, respetando la arquitectura modular del proyecto.

    Clean Architecture
    - Mantén la separación estricta entre `domain`, `application`, `infrastructure`, `handlers`, y `config`.
    - Todo nuevo feature debe dividirse respetando estos límites: 
        - lógica de negocio en `domain/`,
        - casos de uso en `application/`,
        - acceso a AWS (S3, Lambda, etc.) en `infrastructure/`,
        - entrypoints (Lambda handlers) en `handlers/`,
        - bindings de Inversify en `config/`.

    TypeScript
    - Usa interfaces para definir contratos entre capas.
    - Usa clases solo donde sea necesario para lógica de dominio o infraestructura.
    - Usa tipos explícitos, funciones puras y evita duplicación.
    - Escribe código idiomático, conciso y mantenible.

    Serverless / AWS
    - Usa S3 para almacenamiento de archivos y resultados.
    - Usa Lambda para procesamiento asincrónico.
    - Usa URL pre-firmadas para uploads y downloads.
    - Usa eventos de S3 como triggers para procesamiento.
    - Asegura que el código generado esté optimizado para entornos serverless (sin estado, uso de recursos eficiente, arranques rápidos).

    Desarrollo guiado por casos de uso
    - Empieza siempre por definir la entidad y sus operaciones de negocio en `domain/`.
    - Luego crea el caso de uso en `application/` que orquesta la lógica necesaria.
    - Implementa adaptadores de infraestructura en `infrastructure/` para interactuar con S3 u otros servicios.
    - Finalmente, crea handlers en `handlers/` que reciban eventos HTTP o de S3 y llamen a los casos de uso.

    Convenciones y estilo
    - Usa InversifyJS para la inyección de dependencias.
    - Usa Jest para testing. Los tests deben aislar las dependencias con mocks.
    - Usa funciones con el keyword `function` para mantener claridad en el scope.
    - Escribe código orientado a la legibilidad, fácil de testear y mantener.

    Buenas prácticas
    - Separa cálculos, validaciones, y transformaciones del código de infraestructura.
    - Aísla la lógica de negocio de detalles técnicos.
    - Maneja errores de forma controlada, usando Result o errores específicos si aplica.
    - Siempre incluye typings y validaciones al procesar archivos o datos externos.
    - Mantén las funciones puras siempre que sea posible, especialmente en `domain/`.

Cuando el usuario solicite una nueva funcionalidad, genera todo el código necesario (interfaces, casos de uso, adaptadores, handler y bindings) siguiendo la estructura del proyecto. El código debe estar completo, funcional, idiomático y alineado con Clean Architecture y principios SOLID. No dejes TODOs ni placeholders.

---
> Source: [Pesgard/prueba-xid](https://github.com/Pesgard/prueba-xid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
