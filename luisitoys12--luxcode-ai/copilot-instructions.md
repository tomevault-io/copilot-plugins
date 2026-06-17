## luxcode-ai

> Revisa código de interfaz web contra las Web Interface Guidelines de Vercel. Úsalo cuando el usuario pida revisar UI, accesibilidad, UX, animaciones, formularios o rendimiento del frontend.


# Web Interface Guidelines — LuxCode AI Skill

Este skill carga las Web Interface Guidelines de Vercel y las aplica al código
de interfaz de LuxCode AI, especialmente `docs/index.html` y `src/panels/LuxCodePanel.ts`.

## Cómo funciona

1. Obtén las reglas desde la URL fuente arriba
2. Lee los archivos especificados por el usuario (o pregunta cuáles)
3. Aplica todas las reglas del documento obtenido
4. Reporta hallazgos en formato `archivo:línea — problema — sugerencia`

## Archivos principales a revisar en LuxCode AI

- `docs/index.html` — Sitio web público (GitHub Pages)
- `src/panels/LuxCodePanel.ts` — UI del panel en VS Code WebView

## Reglas cargadas desde Vercel Web Interface Guidelines

### 🖱️ Interactividad
- Clic en el `<label>` debe enfocar el `<input>` correspondiente
- Los `<input>` deben estar dentro de `<form>` para submit con Enter
- Los inputs deben tener `type` apropiado: `password`, `email`, `search`, etc.
- Deshabilitar `spellcheck` y `autocomplete` en la mayoría de inputs
- Usar el atributo `required` para validación HTML nativa
- Íconos prefijos/sufijos: posición absoluta encima del input, no al lado
- Toggles deben tener efecto inmediato sin confirmación
- Botones deben deshabilitarse tras submit para evitar doble request
- Elementos interactivos deben tener `user-select: none` en su contenido
- Dropdowns deben abrirse en `mousedown`, no en `click`
- Favicon SVG con `prefers-color-scheme` para modo oscuro/claro
- Elementos solo-ícono deben tener `aria-label` explícito
- Tooltips de hover no deben contener contenido interactivo

### 🎨 Diseño & Feedback
- Actualizar datos optimísticamente, revertir con feedback si falla
- Redirects de auth deben ocurrir en servidor antes de cargar cliente
- Estilizar selección de texto con `::selection`
- Feedback relativo a su trigger:
  - ✅ Mostrar checkmark inline en copy exitoso, no notificación global
  - ✅ Resaltar el input con error, no solo mostrar mensaje genérico
- Skeleton loaders en lugar de spinners para contenido con layout conocido
- Animaciones de entrada con `transform` + `opacity`, no solo `opacity`
- Respetar `prefers-reduced-motion` para todas las animaciones

### ♿ Accesibilidad
- Todos los botones con solo ícono: `aria-label` descriptivo
- Contraste mínimo WCAG AA (4.5:1 texto normal, 3:1 texto grande)
- Navegación completa por teclado: `Tab`, `Enter`, `Escape`, flechas
- Focus visible en todos los elementos interactivos
- Landmarks semánticos: `<nav>`, `<main>`, `<section>`, `<footer>`
- Imágenes decorativas: `alt=""`, imágenes informativas: `alt` descriptivo
- Modales y dialogs: gestión de focus y trap de teclado

### ⚡ Rendimiento
- Fuentes: `font-display: swap` para evitar FOIT
- Imágenes: `loading="lazy"` para imágenes fuera de viewport
- CSS crítico inline en `<head>`, resto diferido
- Evitar layout shifts (CLS): reservar espacio para imágenes/embeds
- Prefetch de rutas probables con `<link rel="prefetch">`

### 📝 Formularios
- Labels siempre visibles (no solo placeholders)
- Mensajes de error específicos por campo, no genéricos
- Estado de carga visual durante submit
- Confirmación de éxito clara y específica
- Autocompletar habilitado donde sea útil (`autocomplete="email"`, etc.)

## Formato de reporte

```
archivo:línea — [CATEGORIA] problema encontrado
  → Sugerencia de corrección
  → Referencia: nombre de la regla
```

## Ejemplo de uso

```
/web-interface-guidelines docs/index.html
```

Resultado:
```
docs/index.html:142 — [ACCESIBILIDAD] Botón de enviar chat sin aria-label
  → Agregar aria-label="Enviar mensaje" al botón .chat-send
  → Referencia: Icon-only interactive elements should define an explicit aria-label

docs/index.html:89 — [ANIMACIÓN] Animación sin prefers-reduced-motion
  → Envolver @keyframes en @media (prefers-reduced-motion: no-preference)
  → Referencia: Respect prefers-reduced-motion for all animations
```

---
> Source: [luisitoys12/luxcode-ai](https://github.com/luisitoys12/luxcode-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
