## markdown-reader

> **Proyecto:** Visor de Markdown Minimalista.

# Guía de Referencia para Gemini Code Assistant

## 1. Resumen del Proyecto
**Proyecto:** Visor de Markdown Minimalista.  

**Descripción:**  
Una aplicación de escritorio instalable para Windows, construida con tecnologías web (HTML, CSS, JavaScript) en un solo archivo.  
El objetivo es proporcionar una herramienta elegante y funcional para visualizar y editar archivos Markdown (`.md`).  
El diseño es minimalista y moderno, enfocado en la usabilidad.

**Nombre de la aplicacion:** Markdown Reader
---

## 2. Contexto y Objetivos
**Aplicación Nativa:**  
El código web debe estar contenido en un único archivo (`main.html`), que será empaquetado como un ejecutable de Windows con **Electron** o **WebView2**.

**Objetivos del MVP (Producto Mínimo Viable):**

- **Fase 1 (Actual):**
  - Crear la estructura básica de `main.html`.
  - Implementar un diseño de doble panel (editor y vista previa).
  - Integrar las librerías necesarias.
  - Habilitar la lectura de archivos Markdown desde el sistema.
  - Sincronizar la vista previa en tiempo real.

- **Fases Futuras:**  
  El plan incluye la adición de funcionalidades como guardar, temas oscuro/claro y optimizaciones de rendimiento.  
  **No implementar estas funcionalidades en esta fase.**

---

## 3. Requisitos de Código y Diseño
- **Archivo Único:**  
  Todo el código (HTML, CSS y JavaScript) debe estar en un solo archivo llamado `main.html`.  
  No se permiten archivos separados.

- **Librerías:**
  - **CSS:** Utilizar Tailwind CSS a través de su CDN.
  - **Markdown:** Usar Showdown.js para la conversión de Markdown a HTML.
  - **Resaltado de Sintaxis:** Usar highlight.js para bloques de código.

- **UI/UX:**
  - **Diseño:** Minimalista, moderno, con enfoque en la funcionalidad.
  - **Disposición:** Dos columnas: un `<textarea>` para la edición a la izquierda y un `<div>` para la vista previa a la derecha.
  - **Tipografía:** Fuente legible como *Inter*.
  - **Controles:** Implementar un botón **"Abrir Archivo"** para el MVP.

---

## 4. Pruebas y Calidad
- **Pruebas de la Fase 1:**  
  La funcionalidad de abrir un archivo y la sincronización en tiempo real deben ser validadas.

- **Manejo de Errores:**  
  Incluir comentarios en el código para el manejo de errores básicos, como la lectura fallida de archivos.

---

## 5. Idioma
Todo el texto visible para el usuario y los comentarios en el código deben estar en **ingles**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ClaudioCeppi83)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ClaudioCeppi83)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
