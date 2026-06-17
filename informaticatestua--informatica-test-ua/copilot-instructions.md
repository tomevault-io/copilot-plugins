## informatica-test-ua

> **DCA Test UA** es una plataforma web interactiva para que estudiantes de Ingeniería Informática de la Universidad de Alicante (UA) practiquen con baterías de preguntas de exámenes y simulacros.

# DCA Test UA — Guía de IA (CLAUDE.md / AGENTS.md)

## 1. Project Overview (Descripción del Proyecto)
**DCA Test UA** es una plataforma web interactiva para que estudiantes de Ingeniería Informática de la Universidad de Alicante (UA) practiquen con baterías de preguntas de exámenes y simulacros. 
Prioriza una experiencia fluida, retroalimentación inmediata, renderizado matemático complejo y soporte para modo oscuro.

## 2. Tech Stack (Tecnologías)
- **Framework**: Astro 5 (Generación Estática + Rutas Dinámicas).
- **Core Logic**: Vanilla JS en el cliente (`main.js`).
- **Styling**: Tailwind CSS 4 + CSS nativo.
- **Math & Code**: KaTeX (fórmulas LaTeX) y Prism.js (resaltado de código).
- **Deployment**: Vercel.

## 3. Architecture Rules (Reglas de Arquitectura)
- **Motor Mono-Archivo (`main.js`)**: Maneja carga asíncrona de `.txt`, parseo, shuffle y validación.
- **Estado Persistente**: `estadosPreguntas` guarda selecciones de usuario para poder navegar entre preguntas sin pérdida de información.
- **Data Protocol**: Preguntas almacenadas en `public/resources/data/*.txt`. Soporte nativo para Markdown, código (`` ` ``), LaTeX (`$$ ... $$`) e imágenes `![Alt](URL){width=X height=Y}`.
- **Rutas Dinámicas**: `src/pages/[subject].astro` inyecta el ID de la asignatura de la URL al motor JS.

## 4. File/Directory Structure (Estructura de Archivos)
- `public/resources/data/`: Datasets de preguntas en formato texto (`.txt`).
- `public/resources/js/main.js`: Lógica central del sistema de cuestionarios.
- `src/layouts/BaseLayout.astro`: Layout maestro (gestión del `<head>` y scripts globales).
- `src/pages/`: Rutas de la aplicación web (incluye `index.astro` y las páginas dinámicas).
- `src/styles/global.css`: Variables globales de CSS y configuración base de Tailwind.

## 5. Coding Conventions (Convenciones de Código)
- **Nomenclatura**: Los archivos de datos deben llamarse `[id-asignatura]Preguntas.txt` (las excepciones van explícitamente en `main.js`).
- **UI/UX y Estilos**: Usa clases utilitarias de Tailwind respetando las variables semánticas (surface, background, text-main) definidas en `global.css`.
- **Accesibilidad**: Mantener e impulsar la navegación por teclado (teclas 1-5, Enter, etc.) en todos los componentes interactivos.
- **Rendimiento**: Preferir etiquetas `<script is:inline>` para librerías de terceros, evitando hinchar el bundle de Astro.

## 6. "Never" Section (Lo que NUNCA debes hacer)
- **NUNCA** introduzcas frameworks JS reactivos (React, Vue, Svelte) para la lógica core del cuestionario; mantén estricto Vanilla JS.
- **NUNCA** alteres el formato posicional y de salto de línea de los archivos `.txt` de preguntas.
- **NUNCA** utilices colores "hardcodeados" en el HTML que rompan la paleta y el funcionamiento del modo oscuro.
- **NUNCA** asumas que el proyecto usará renderizado en el servidor (SSR) para la lógica pesada; Astro opera aquí como Generador de Sitios Estáticos (SSG).

## 7. Specific Commands (Comandos Específicos)
- `npm install` - Instalar las dependencias del proyecto.
- `npm run dev` - Arrancar el servidor de desarrollo en `localhost:4321`.
- `npm run build` - Generar la compilación estática del proyecto (carpeta `dist/`).
- `npm run preview` - Previsualizar localmente la compilación de producción.

## 8. Pointers to Reference Files (Puntos de Referencia)
- **Documentación de Usuario e Instalación**: Consulta `README.md`.
- **Sistema de Diseño y Paletas**: Consulta `src/styles/global.css`.

---
> Source: [informaticatestua/Informatica-Test-UA](https://github.com/informaticatestua/Informatica-Test-UA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
