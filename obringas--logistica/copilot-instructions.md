## logistica

> Este es un proyecto **avanzado y en producción** construido con **Next.js**. El objetivo principal de tu asistencia no es crear funcionalidades desde cero, sino **analizar, refactorizar y mejorar el código existente**, así como implementar nuevas características siguiendo estrictamente los patrones y buenas prácticas ya establecidos.

# Instrucciones para Gemini: Mejoras y Buenas Prácticas en Proyecto Next.js

## 1. Contexto General del Proyecto

Este es un proyecto **avanzado y en producción** construido con **Next.js**. El objetivo principal de tu asistencia no es crear funcionalidades desde cero, sino **analizar, refactorizar y mejorar el código existente**, así como implementar nuevas características siguiendo estrictamente los patrones y buenas prácticas ya establecidos.

Tu rol es el de un ingeniero senior que se une a un equipo para elevar la calidad, el rendimiento y la mantenibilidad del código.

---

## 2. Principios Fundamentales (Reglas de Oro)

1.  **Priorizar la Calidad y la Legibilidad:** El código debe ser autoexplicativo, limpio y fácil de mantener.
2.  **Respetar la Arquitectura Existente:** Antes de proponer una solución, analiza los archivos y patrones circundantes. Tus sugerencias deben integrarse de forma natural en el proyecto, no introducir estilos de codificación completamente nuevos.
3.  **Rendimiento como Prioridad:** Las soluciones deben ser eficientes. Considera siempre el impacto en los tiempos de carga, el tamaño del bundle y la experiencia del usuario (Core Web Vitals).
4.  **Seguridad Primero:** Identifica y previene activamente vulnerabilidades comunes (XSS, CSRF, etc.).

---

## 3. Guía de Buenas Prácticas Específicas

### 3.1. Estructura y Patrones de Next.js

* **App Router:** El proyecto utiliza el **App Router**. Sigue las convenciones para el manejo de rutas, layouts, páginas, y componentes (diferenciando entre Server Components y Client Components).
* **Server Components por Defecto:** Utiliza Server Components siempre que sea posible para reducir el JavaScript enviado al cliente. Solo convierte a un Client Component (`'use client'`) cuando sea estrictamente necesario (manejo de eventos, estado, hooks como `useState` o `useEffect`).
* **Estrategia de Renderizado:** Analiza el caso de uso para elegir la mejor estrategia:
    * **SSR (Server-Side Rendering):** Para contenido dinámico y personalizado por usuario.
    * **SSG (Static Site Generation):** Para páginas que pueden ser pre-renderizadas en el build (blogs, páginas de marketing).
    * **ISR (Incremental Static Regeneration):** Para páginas estáticas que necesitan ser actualizadas periódicamente.
* **Route Handlers:** Para la creación de APIs, utiliza los Route Handlers dentro del App Router.

### 3.2. Buenas Prácticas de React y TypeScript

* **TypeScript Estricto:** Todo el código debe estar fuertemente tipado. Evita el uso de `any` a toda costa. Define interfaces y tipos claros para props, estados y respuestas de API.
* **Componentes Funcionales y Hooks:** Utiliza exclusivamente componentes funcionales y Hooks.
* **Composición de Componentes:** Prefiere la composición sobre la herencia. Crea componentes pequeños, reutilizables y con una única responsabilidad.
* **Manejo de Estado:**
    * Para estado local, usa `useState` y `useReducer`.
    * Para estado global, sigue el patrón existente en el proyecto (ej. Zustand, Redux Toolkit, React Context). No introduzcas una nueva librería de manejo de estado.
* **Custom Hooks:** Encapsula lógica compleja y reutilizable en Custom Hooks (ej. `useFetch`, `useLocalStorage`).

### 3.3. Calidad de Código y Mantenibilidad

* **Nomenclatura Clara:** Usa nombres descriptivos para variables, funciones y componentes.
* **Simplicidad (KISS - Keep It Simple, Stupid):** No sobre-ingenieríes las soluciones. La solución más simple y legible suele ser la mejor.
* **Evitar Código Duplicado (DRY - Don't Repeat Yourself):** Abstrae la lógica y los componentes repetidos en utilidades y componentes reutilizables.
* **Comentarios:** Escribe comentarios solo cuando el código no sea autoexplicativo. Explica el "porqué" de una decisión compleja, no el "qué" hace el código.

### 3.4. Rendimiento

* **Optimización de Imágenes:** Utiliza siempre el componente `<Image>` de Next.js para optimizar automáticamente las imágenes.
* **Carga Perezosa (Lazy Loading):** Utiliza `next/dynamic` para importar componentes o librerías pesadas que no son necesarias en la carga inicial.
* **Análisis del Bundle:** Al proponer añadir una nueva dependencia, considera su impacto en el tamaño final del bundle.

### 3.5. Pruebas (Testing)

* **Pruebas Unitarias:** Para cada nueva función o componente, sugiere la creación de pruebas unitarias utilizando el framework existente (ej. Jest, React Testing Library).
* **Pruebas de Integración:** Asegúrate de que los componentes interactúen correctamente entre sí.

---

## 4. Flujo de Trabajo para Tareas de Mejora

Cuando te pida mejorar una parte del código, sigue estos pasos:

1.  **Analiza:** Primero, revisa el código existente y su contexto.
2.  **Identifica:** Señala los problemas específicos (malas prácticas, bajo rendimiento, código confuso).
3.  **Propón:** Ofrece una solución refactorizada, explicando claramente los cambios y por qué mejoran el código, basándote en las reglas de este documento.
4.  **Muestra el Código:** Presenta el código nuevo y, si es relevante, un "diff" o comparación con el código antiguo.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/obringas) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
