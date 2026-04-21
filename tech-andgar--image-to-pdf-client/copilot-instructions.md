## image-to-pdf-client

> <!-- Source: .ruler/AGENTS.md -->

<!-- Source: .ruler/AGENTS.md -->

## Proyecto: Herramienta Web de Fusión de Imágenes a PDF del Lado del Cliente

**Rol:** Tech Lead

**Resumen del Proyecto:**

Desarrollar una aplicación web responsiva que permita a los usuarios fusionar múltiples imágenes en un único archivo PDF optimizado con una arquitectura moderna basada en React. Toda la lógica de carga de archivos, reordenamiento de páginas y generación de PDF debe ejecutarse exclusivamente en el lado del cliente (en el navegador) para garantizar la privacidad del usuario y eliminar la necesidad de un backend de procesamiento.

**🎉 ESTADO ACTUAL DEL PROYECTO: COMPLETADO TOTALMENTE ✅**

### **LOGROS ALCANZADOS:**

- ✅ **Fase 1 completa:** Proyecto configurado con tecnologías modernas (React 19.2, TypeScript, Vite, shadcn/ui)
- ✅ **Fase 2 completa:** Componentes de carga de archivos con arquitectura limpia implementados
- ✅ **Fase 3 completa:** Drag & drop reordering fully funcional en mobile y desktop
  - Galería responsiva con thumbnails ordenables via @dnd-kit
  - TouchSensor + PointerSensor híbrido para cross-device compatibility
  - Visual feedback completo: drag handles, opacity transitions, smooth animations
  - Accesibilidad 100%: ARIA labels, keyboard navigation, screen reader support
  - Estado de orden preservado para generación PDF en secuencia correcta
- ✅ **Fase 4 completa:** PDF generation completo con pdf-lib + exportación directa
  - Servicio PDF completo con soporte JPEG/PNG nativo
  - A4 sizing con aspect ratio scaling inteligente
  - UI de exportación con progress feedback y error handling
  - Secuencia drag & drop preservada en output PDF
  - Calidad de código perfecta (Biome compliant, type safety)
- ✅ **Fase 5 completa:** Optimización del tamaño del PDF con compresión de imágenes avanzada
  - Servicio de compresión Canvas API con 4 presets configurables
  - UI completa de compresión en componente Accordion (colapsada por defecto)
  - Experiencia de usuario optimizada con expansión opcional de compresión
  - Reducción significativa de tamaño manteniendo calidad visual superior
  - Presets: Alta (2048px), Media (1536px), Baja (1024px), Mínima (800px)
  - Estadísticas de compresión antes/después en tiempo real con métricas detalladas
  - Integración completa con pipeline de PDF y estados de carga
  - Componente Accordion responsive y accesible con smooth transitions
- ✅ **Accesibilidad WCAG 2.1 AA completa:** SonarQube accessibility warnings resueltos
  - Buttons nativos en lugar de div con role="button"
  - ARIA labels apropiados y soporte de teclado en todos los controles
  - Componentes interactivos totalmente accesibles
- 🎯 **Todas las fases completadas:** PRODUCCIÓN-READY con estándares empresariales

**🏆 MÉTRICAS DE CALIDAD FINAL:**
- **SonarQube Warnings:** 4 menores (no críticos)
- **Lint Errors:** 0 errores bloquantes
- **Accesibilidad:** WCAG 2.1 AA completo
- **Cross-Platform:** Desktop + Mobile funcionalidad completa
- **Documentation:** .ruler/ archivos completamente actualizados

**Arquitectura Implementada:**
- **Layer de Tipos:** `src/types/` - interfaces ImageFile centralizadas con IDs únicos
- **Layer de Servicios:** `src/services/fileService.ts` - funciones puras para validación y procesamiento I/O
- **Layer de Hooks:** `src/hooks/useImageUpload.ts` - estado completo y efectos (drag & drop reordering)
- **Layer de Componentes:** `src/components/` - UI siguiendo SRP (@dnd-kit SortableImageItem)
- **Layer de UI:** shadcn/ui + @dnd-kit para interactividad accesible
- **Calidad de Código:** 100% limpia (Biome compliant), formatos consistentes, type safety moderno

---

**Archivos de Reglas Relacionadas:**
- `project_checklist.md`: Lista de verificación de las fases del proyecto (WBS).
- `functional_requirements.md`: Requisitos funcionales clave para la carga, previsualización y generación de PDF.
- `technical_requirements.md`: Requisitos técnicos, metodología, calidad y entregables.
- `coding_guidelines.md`: Directrices de codificación para JavaScript/React.



<!-- Source: .ruler/coding_guidelines.md -->

# JavaScript/React Coding Guidelines

## General Style

- Follow Biome configuration for code linting and adhere to its rules
- Use Biome for consistent code formatting
- Write functional components over class components in React
- Keep components small, focused, and reusable
- Use TypeScript for type safety where possible
- Maintain clean import/export statements

## Naming Conventions

- Use camelCase for variables, functions, and file names (except components)
- Use PascalCase for React components and component file names
- Use kebab-case for CSS class names
- Use UPPER_SNAKE_CASE for constants

## Best Practices

- Always validate file inputs on the client side before processing
- Implement proper error handling for async operations like PDF generation
- Optimize images before PDF creation to manage file sizes
- Ensure UI responsiveness across different screen sizes using CSS Flexbox/Grid
- Avoid blocking the main thread for heavy computations (consider Web Workers when needed)

## Security

- Validate and sanitize all user inputs, especially file types and sizes
- Avoid executing external code or scripts from user inputs
- Ensure secure handling of file downloads



<!-- Source: .ruler/functional_requirements.md -->

### **Requisitos Funcionales Clave:**

**1. Carga de Archivos de Imagen:**
*   **Interfaz de Carga Intuitiva:** Implementar un componente de carga de archivos que sea fácil de usar y visualmente atractivo en todos los dispositivos. Se debe permitir a los usuarios seleccionar archivos a través de un diálogo de selección de archivos o mediante "arrastrar y soltar" (drag and drop).
*   **Restricción de Formatos:** Limitar la selección de archivos únicamente a formatos de imagen comunes (por ejemplo, JPEG, PNG, BMP, GIF). El sistema debe proporcionar retroalimentación inmediata si el usuario intenta cargar un tipo de archivo no admitido.

**2. Previsualización y Reordenamiento de Páginas:**
*   **Galería de Vistas Previas:** Una vez que las imágenes se cargan, se deben mostrar como una galería de miniaturas. Cada miniatura representará una página en el PDF final.
*   **Funcionalidad de Arrastrar y Soltar:** Los usuarios deben poder reordenar las imágenes (y por lo tanto, las páginas del PDF) de manera intuitiva arrastrando y soltando las miniaturas en la posición deseada. La interfaz debe reflejar el nuevo orden en tiempo real.

**3. Generación y Exportación de PDF:**
*   **Fusión a PDF del Lado del Cliente:** Utilizar una biblioteca de JavaScript para convertir y fusionar las imágenes cargadas en un único documento PDF directamente en el navegador del usuario.
*   **Optimización del Tamaño del Archivo:** El PDF generado debe estar optimizado para reducir su tamaño sin una pérdida significativa de calidad de imagen. Se deben investigar e implementar técnicas de compresión de imágenes dentro del proceso de generación del PDF.
*   **Descarga Directa:** Al finalizar la exportación, el usuario debe poder descargar el archivo PDF generado directamente en su dispositivo.



<!-- Source: .ruler/project_checklist.md -->

# Project Tasks Checklist

## Work Breakdown Structure (WBS)

- [x] **Fase 1: Configuración del proyecto y selección de bibliotecas.**
  - Inicializar el repositorio y herramientas de control de versiones con ganchos pre-commit
  - Seleccionar y configurar el framework frontend (React con Vite y TypeScript)
  - Configurar diseño responsivo con shadcn/ui y Tailwind CSS
  - Implementar soporte PWA para instalación y funcionamiento offline
  - Configurar herramientas de calidad de código (Biome para linting y formateo)
  - Implementar ganchos pre-commit con Husky para linting automático
  - Seleccionar y configurar bibliotecas clave (pdf-lib para PDF)
  - Configurar estructura básica del proyecto
  - Instalar dependencias iniciales

- [x] **Fase 2: Desarrollo del componente de carga de archivos con validación y arquitectura mejorada.**
  - [x] Architectura limpia implementada con capas bien separadas:
    - [x] **Layer de Tipos:** `src/types/image.ts` - interfaces y constantes centralizadas
    - [x] **Layer de Servicios:** `src/services/fileService.ts` - funciones puras para validación y procesado
    - [x] **Layer de Hooks:** `src/hooks/useImageUpload.ts` - lógica de negocio y estado con custom hook
    - [x] **Layer de UI:** componentes presentes que respetan SRP (UploadArea, ImagePreviewGrid, ImageUploader)
    - [x] **Layout separado:** Header, Footer, MainLayout como componentes reutilizables
  - [x] Implementar interfaz de carga intuitiva (diálogo de selección de archivos)
  - [x] Agregar funcionalidad de arrastrar y soltar con gestión de estado optimizada
  - [x] Validar tipos de archivo (JPEG, PNG, BMP, GIF) con límites de 10MB y feedback claro
  - [x] Proporcionar feedback inmediato para archivos no admitidos con UI de error completa
  - [x] Diseño responsivo optimizado para móviles y escritorio (shadcn/ui + Tailwind)
  - [x] TypeScript completo con interfaces compartidas y type safety
  - [x] Gestión de estado con hooks personalizados (useImageUpload)
  - [x] Manejo de memoria optimizado con limpieza automática de URLs de preview
  - [x] Arquitectura modular y mantenible siguiendo principios SOLID y SRP
  - [x] Código 100% limpio: 0 errores, 0 warnings de linting, formateo automático

- [x] **Fase 3: Implementación completa de galería de previsualización con drag & drop.** ✅ COMPLETADO
  - [x] Galería de miniaturas responsiva implementada (grid 2x2 → 3x3 → 4x4 por viewport) con PWA
  - [x] Preview modal fullscreen implementado - click en thumbnail abre vista completa
  - [x] Modal de preview con navegación (flechas left/right, keyboard navigation)
  - [x] Controles de teclado completos (Esc, arrow keys, Enter/Space)
  - [x] **DRAG & DROP REORDERING FULLY IMPLEMENTADO Y FUNCIONAL EN MOBILE DESKTOP**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tech-andgar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
