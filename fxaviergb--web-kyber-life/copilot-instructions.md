## web-kyber-life

> Este archivo actúa como un orquestador para el asistente de IA (Antigravity). Define qué **skills** deben cargarse y aplicarse obligatoriamente según el tipo de tarea que se esté realizando en este repositorio (`web-kyber-life`).

# AI Agents & Skills Configuration (AGENTS.md)

Este archivo actúa como un orquestador para el asistente de IA (Antigravity). Define qué **skills** deben cargarse y aplicarse obligatoriamente según el tipo de tarea que se esté realizando en este repositorio (`web-kyber-life`).

## 🧠 Instrucción Principal para la IA
**A LA IA:** Antes de comenzar a escribir código o planificar una solución, DEBES identificar la categoría de la tarea en la tabla inferior y **leer los archivos `SKILL.md` correspondientes** usando tus herramientas. Aplica estrictamente las convenciones, patrones y restricciones definidas en esos skills. Si tienes dudas, pregunta antes de codificar.

---

## 🛠️ Mapeo de Tareas a Skills

### 1. Desarrollo Frontend (React & Next.js)
Cuando la tarea involucre crear páginas, rutas, hooks, o lógica de UI en React/Next.js:
- **`nextjs-best-practices`**: Para arquitectura de App Router, Server/Client components y data fetching.
- **`react-patterns`**: Para el uso correcto de hooks, composición y tipado fuerte.
- **`react-ui-patterns`**: Para manejar estados de carga (loading), manejo de errores y estados vacíos de forma elegante.

### 2. Diseño UX/UI (Estética y Estilos)
Cuando la tarea sea puramente visual, maquetación, o creación de componentes visuales de alto impacto:
- **`ui-skills`**: (CRÍTICO) Aplica restricciones de diseño premium, micro-interacciones y acabados de alta fidelidad. No usar diseños genéricos o básicos.
- **`tailwind-patterns`**: Para utilizar clases utilitarias correctamente y mantener consistencia.
- **`radix-ui-design-system`**: Si se necesita crear componentes base accesibles (botones, modales, menús accesibles).

### 3. Calidad, Arquitectura y Refactorización
Cuando se solicite limpiar código, mejorar rendimiento o definir arquitectura:
- **`clean-code`**: Para asegurar código directo, pragmático y fácil de mantener.
- **`coding-standards`**: Para asegurar convenciones universales de la industria en JS/TS.
- **`web-performance-optimization`**: Si hay que revisar tiempos de carga, Core Web Vitals, o tamaño de bundles.

### 4. Seguridad y Autenticación
Si se toca el flujo de login, protección de rutas, validación de inputs o creación de APIs:
- **`security-review`**: Checklists de seguridad obligatorios antes de escribir funciones expuestas.
- **`nextjs-supabase-auth`**: Si se realizan integraciones o ajustes con Supabase Auth, seguir estas reglas específicas.

---

## 📝 Reglas Generales y Filosofía del Proyecto
1. **Nunca sacrifiques la estética**: Si haces un cambio visual en la UI, asegúrate de que mantenga la identidad del proyecto y se vea profesional.
2. **TypeScript Estricto**: Todo código nuevo debe estar debidamente tipado. Prohibido usar `any` a menos que sea estrictamente necesario.
3. **Pragmatismo sin Sobre-Ingeniería**: Mantén el código lo más simple posible que resuelva de manera robusta el problema actual, siguiendo el espíritu de `clean-code`.
4. **Diseño Responsivo Obligatorio (Mobile-First)**: Todo componente o vista que se modifique o cree debe verse y funcionar perfectamente en todos los tamaños de pantalla (móviles, tablets y escritorio) garantizando una experiencia fluida.

---
> Source: [fxaviergb/web-kyber-life](https://github.com/fxaviergb/web-kyber-life) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
