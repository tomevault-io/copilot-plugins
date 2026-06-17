## weather-station-esp

> Este skill guía al usuario para crear, organizar y mantener una Wiki técnica dentro del repositorio, asegurando que la documentación sea clara, útil y visible en la sección Wiki de GitHub.

# Skill: Crear y mantener una Wiki en el repositorio

## Descripción
Este skill guía al usuario para crear, organizar y mantener una Wiki técnica dentro del repositorio, asegurando que la documentación sea clara, útil y visible en la sección Wiki de GitHub.

## Proceso paso a paso
1. **Planificación**
   - Identifica los temas clave y estructura la Wiki (introducción, instalación, uso, troubleshooting, FAQ, etc.).
   - Define responsables y convenciones de formato (Markdown, enlaces internos, imágenes, etc.).
2. **Creación inicial**
   - Crea la carpeta `/wiki` en el repositorio (si se prefiere mantener la wiki en el repo y luego sincronizarla con la Wiki de GitHub).
   - Redacta los archivos principales: `Home.md`, `Instalacion.md`, `Uso.md`, etc.
   - Usa títulos claros y enlaces entre páginas.
3. **Publicación en GitHub Wiki**
   - Si la Wiki de GitHub está habilitada, sube los archivos a la sección Wiki usando la interfaz web o sincroniza vía git (`git clone https://github.com/OWNER/REPO.wiki.git`).
   - Verifica que la navegación y los enlaces funcionen correctamente.
4. **Mantenimiento**
   - Actualiza la Wiki con cada cambio relevante en el proyecto.
   - Revisa enlaces rotos, ortografía y consistencia.
   - Fomenta la colaboración y revisa contribuciones externas.

## Decisiones y ramificaciones
- ¿La Wiki se mantiene solo en GitHub o también en el repo como `/wiki`?
- ¿Se automatiza la sincronización entre `/wiki` y la Wiki de GitHub?
- ¿Qué convenciones de formato y estructura se siguen?

## Criterios de calidad
- Navegación clara y enlaces funcionales.
- Documentación actualizada y relevante.
- Uso consistente de formato y estructura.
- Accesibilidad desde la sección Wiki de GitHub.

## Ejemplo de prompts
- "Crea la estructura inicial de la Wiki para este repo."
- "Sincroniza la carpeta /wiki con la Wiki de GitHub."
- "Sugiere secciones clave para la documentación del proyecto."
- "¿Cómo puedo automatizar la actualización de la Wiki?"

## Sugerencias relacionadas
- Skill para automatizar la sincronización entre `/wiki` y la Wiki de GitHub.
- Skill para validar enlaces y estructura de la Wiki.
- Skill para generar documentación técnica a partir de comentarios en el código.

---
> Source: [alepape1/weather-station-ESP](https://github.com/alepape1/weather-station-ESP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
