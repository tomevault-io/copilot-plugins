## pokedb

> Estas directrices están diseñadas para que los agentes de IA sean productivos y mantengan la coherencia del proyecto.



# � Instrucciones para Agentes de IA

Estas directrices están diseñadas para que los agentes de IA sean productivos y mantengan la coherencia del proyecto.

## 🏗️ Arquitectura y Patrones Clave



## ⚙️ Flujos de Desarrollo

 - **Instalación**: `bun install` y `bun run dev` para desarrollo local.

- **Actualización de dependencias**: Usa `bun upgrade` para actualizar todas las dependencias o `bun add <paquete>` para agregar nuevas. Tras actualizar, revisa que la app funcione correctamente y no haya advertencias en consola.

- **Commits**: Usa mensajes alegres y descriptivos, con emojis y lista de cambios clave. Ejemplo:
  ```
  feat(checklist): ✨ Sistema completo de checklist con adaptación automática de temas 🎨

  ### 📝 Cambios principales:
  * 🔄 Refactorización completa del componente ChecklistSolid con detección automática de tema
  * 🎨 Implementación de estilos dinámicos que se adaptan en tiempo real a modo claro/oscuro
  * 📱 Toolbar responsivo con filtros (todos/completados/pendientes) y búsqueda
  * ➕ Sistema CRUD completo: añadir, editar y eliminar ítems del checklist
  * 💾 Persistencia en localStorage para checkboxes y ítems editables
  * 🎭 Modal de edición con componente Show de SolidJS para renderizado condicional
  * 🎪 MutationObserver para detectar cambios de tema automáticamente
  * 📐 Funciones helper dinámicas: getCardStyles(), getToolbarStyles(), getInputStyles(), getTextClasses()
  * 🌈 Gradientes de texto compatibles con bg-clip-text en ambos temas
  * 🛠️ Mejoras en CSS global para soporte completo de modo claro

  ### 🚀 Funcionalidades implementadas:
  * Filtrado inteligente por estado de completado y búsqueda de texto
  * Añadir nuevos ítems con formulario inline
  * Editar ítems existentes con modal dedicado  
  * Eliminar ítems con confirmación visual
  * Contadores dinámicos de ítems por categoría
  * Transiciones suaves entre temas sin parpadeos
  * Diseño completamente responsivo para móvil y desktop

  ### 🎯 Arquitectura:
  * SolidJS signals para gestión de estado reactivo
  * Componentes modulares con separación de responsabilidades  
  * Patrones de persistencia con localStorage
  * Sistema de temas centralizado y automático"
  ```
- **Push**: Tras cada commit importante, pregunta si se debe subir a remoto.
- **Refactorización**: Antes de modificar, analiza el contexto y busca patrones existentes.
- **Migración de Componentes**: Si migras componentes interactivos, sigue la estrategia de "wrapper .astro" y usa `client:visible` como directiva preferida.

## � Depuración y Resolución de Problemas

- Explica los errores y sugiere soluciones claras.
- Para errores de hidratación en Astro, sigue el enfoque de wrapper y directiva `client:visible`.

## 📚 Documentación y Buenas Prácticas

- Explica conceptos técnicos con ejemplos del proyecto.
- Genera documentación clara y útil en los cambios relevantes.
- Mantén la accesibilidad, el SEO y el rendimiento óptimo como prioridades.

## 🔄 Proceso de Modificación de Archivos

Antes de realizar cualquier cambio en un archivo existente, sigue este proceso:

1.  **Lectura y Comprensión**: Lee el contenido completo del archivo para entender su propósito, lógica, dependencias y cómo se integra en el proyecto. Presta atención a los comentarios, la estructura y los patrones de código existentes.
2.  **Análisis de Requisitos**: Revisa los cambios solicitados o la tarea a realizar. Identifica las secciones específicas del archivo que se verán afectadas y cómo la modificación impactará el comportamiento general.
3.  **Propuesta de Solución**: Formula una solución clara y concisa. Considera las mejores prácticas, la coherencia con el código existente y la eficiencia. Si hay múltiples enfoques, evalúa sus pros y contras.
4.  **Implementación**: Aplica los cambios propuestos. Asegúrate de mantener la indentación, el formato y la sintaxis correctos. Si es necesario, añade comentarios para explicar la nueva lógica o las decisiones de diseño.
5.  **Verificación**: Una vez realizados los cambios, verifica que la aplicación funcione correctamente y que los cambios se hayan aplicado según lo esperado. Realiza pruebas si es necesario.

## 🗂️ Estructura del Proyecto
- `src/components/` — Componentes UI reutilizables (Astro)
- `src/pages/` — Páginas principales y rutas dinámicas
- `public/` — Assets estáticos e iconos
- `docs/` — Documentación técnica

---
Estas reglas resumen los patrones y convenciones reales del proyecto. Si algo no está claro o falta, pregunta antes de actuar.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NachoSizle)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/NachoSizle)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
