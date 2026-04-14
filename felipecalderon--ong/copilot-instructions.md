## ong

> Este proyecto es una plataforma web para que ciudadanos denuncien maltrato animal y delitos medioambientales. El objetivo es centralizar información y facilitar la acción de organizaciones y autoridades.

# Contexto del Proyecto: Plataforma de Denuncia Animal y Medioambiental

Este proyecto es una plataforma web para que ciudadanos denuncien maltrato animal y delitos medioambientales. El objetivo es centralizar información y facilitar la acción de organizaciones y autoridades.

Palabras clave importantes en este dominio incluyen: "denuncia", "maltrato animal", "delito medioambiental", "evidencia", "seguimiento", "autoridades competentes".

## Tech Stack

El proyecto utiliza las siguientes tecnologías (Monolito de Nextjs):

-   **Frontend:** Next.js (App Router), React, TypeScript, Tailwind CSS, Shadcn/UI, Zustand (gestión de estado).
-   **Backend:** Next.js (Server Actions, API Routes), Node.js, Mongoose.
-   **Base de Datos:** MongoDB.
-   **Autenticación:** NextAuth.js.cv

Prioriza soluciones y ejemplos de código que sean nativos o compatibles con este stack.

## Estructura del Proyecto

La estructura principal del proyecto es:
/src
├───actions/ # Lógica de servidor (Server Actions)
├───app/ # Rutas, páginas, layouts (App Router)
│ ├───api/ # Rutas de API
│ ├───posts/ # Gestión de posts/denuncias
│ └───perfil/ # Gestión de información personal
├───components/ # Componentes React (UI y personalizados)
│ └───ui/ # Componentes base de Shadcn/UI
├───database/ # Conexión DB y modelos Mongoose
│ └───models/ # Esquemas de Mongoose
├───hooks/ # Hooks de React personalizados
├───interfaces/ # Tipos y interfaces TypeScript
├───lib/ # Funciones de utilidad y helpers
├───providers/ # Proveedores de contexto React
└───stores/ # Stores de Zustand

Cuando se te pida crear nuevos archivos o componentes, considera esta estructura para su ubicación lógica.

## Estándares de Código

-   Utiliza TypeScript estrictamente, define interfaces explícitamente para los datos.
-   Sigue las convenciones de nombres de Next.js versión 15 para rutas y archivos.
-   Preferencia por React Server Components y Server Actions para lógica de backend.
-   Los componentes de UI deben seguir la convención de Shadcn/UI y ser lo más reusables posible.
-   Usa camelCase para variables y funciones, PascalCase para componentes y tipos.
-   Incluye comentarios claros para lógica compleja o decisiones de diseño.

## Directrices para Gemini

-   Conciso y directo en las respuestas.
-   Al refactorizar, explica brevemente el porqué de los cambios.
-   Siempre valida las soluciones con la estructura del proyecto y el tech stack definidos.

## Patrones Comunes

-   Para la persistencia de datos, siempre interactúa a través de modelos de Mongoose definidos en `src/database/models`.
-   Las acciones de usuario que modifiquen datos deben implementarse como Next.js Server Actions.
-   La autenticación se maneja exclusivamente con NextAuth.js. Si se requiere proteger una ruta, usa `useSession`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/felipecalderon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/felipecalderon)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
