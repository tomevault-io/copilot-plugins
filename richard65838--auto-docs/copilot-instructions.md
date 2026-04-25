## auto-docs

> description: Detecta los subproyectos en el workspace, genera documentación específica para cada uno mediante reglas por framework (creándolas si no existen adaptadas al stack), y construye una visión general del proyecto en auto-docs/project_summary.md.

name: auto-docs
description: Detecta los subproyectos en el workspace, genera documentación específica para cada uno mediante reglas por framework (creándolas si no existen adaptadas al stack), y construye una visión general del proyecto en auto-docs/project_summary.md.
antes de comenzar lee la documentacion manual en la carpeta raiz "documentation/"
command: @auto-docs

# Este es el FLUJO OBLIGATORIO que siempre debe seguir
Antes de comenzar lee la documentacion en auto-docs/project_summary.md si existe.

flowchart TD
    A["Si no existe la carpeta auto-docs creala"] --> A1["La raiz es un repositorio?"]
    A1 -- No -->  B2["Detecta los proyectos y frameworks"]
    A1 -- Si --> B["Detecta el framework del repositorio en nombre del repositorio es el nombre de la carpeta raiz"]
    B --> C["Itera sobre los proyectos encontrados, y los aborda uno a uno"]
    B2 --> C
    C --> D{"Existe una regla project-documentation-agents para el framwork del proyecto ubicada en cursor/rules/documentation/project-documentation-agents tiene reglas como: 
     - rails-agent.mdc
     - react-agent.mdc
    "}
    D -- No --> E["Analiza qué ítems son necesarios para la documentación del framework"]
    E --> F["Genera una regla project-documentation-agents para la documentación del framework almacenala en cursor/rules/documentation/project-documentation-agents/"]
    F --> G["Utiliza la project-documentation-agents del framework para generar la documentación en sub-project.md"]
    D -- Sí --> G
    G --> H{"¿Existe una documentación del proyecto actualmente?"}
    H -- Sí --> I["Leer la documentación"]
    I --> J["Mejorar o crear la documentación"]
    H -- No --> J
    J --> M["Verificar que la documentación creada o modificada almacenada en auto-docs"]
    M --> N["Corregir cualquier referencia a elementos inexistentes"]
    N --> K{"¿Hay más proyectos?"}
    K -- Sí --> C
    K -- No --> L["Genera el documento auto-docs/project-summary.md"]
    L --> O["Realizar verificación final de toda la documentación para confirmar su exactitud"]


## Archivo:
auto-docs/project_summary.md: Es la documentacion general
Incluye:
- Arquitectura
- Descripcion general del proyecto
- Descripcion breve de los sub proyectos  (frameworks y descripcion breve)

No incluye:
- Configuraciones iniciales
- Ciclo de desarrollo
- Configuraciones
- Herrambientas en el workspace

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/richard65838) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
