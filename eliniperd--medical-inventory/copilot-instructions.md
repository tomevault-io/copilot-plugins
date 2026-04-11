## medical-inventory

> Este es un proyecto construido con **Next.js 14.2.12** utilizando el **App Router**.

# Contexto de la Aplicación Next.js 14

## Arquitectura del Proyecto
Este es un proyecto construido con **Next.js 14.2.12** utilizando el **App Router**.

-   **Lenguaje:** TypeScript (`.jsx` y `.js`).
-   **Estilo:** Funciones de componentes con la sintaxis de **Arrow Functions**.
-   **Estilizado:** Utilizamos **Tailwind CSS** para los estilos. Los componentes deben usar `className` y no estilos en línea.
## -   **Gestión de Estado:** Se usa `zustand` para el estado global y `useReducer` para el estado local complejo.

## Estructura de Directorios Clave

-   **`app/`**: Contiene todas las rutas y *layouts* de la aplicación. Todos los componentes de esta carpeta son **Server Components** por defecto, a menos que se use la directiva `'use client'`.
-   **`components/`**: Componentes reutilizables. Si no tienen lógica de React que requiera estado o efectos (hooks), deben ser **Server Components** simples.
-   **`lib/`**: Funciones de utilidad, lógica de negocio y archivos de conexión a la base de datos (secreto y no deben ser importados en componentes cliente).
-   **`public/`**: Assets estáticos (imágenes, fuentes, etc.).
-   **`prisma/`**: schema de la base de datos

## Directivas Específicas para Gemini

-   **Rendimiento:** Siempre que sea posible, prefiere los **Server Components** para el trabajo pesado y la obtención de datos para mejorar el rendimiento inicial. Usa `'use client'` solo si es estrictamente necesario (interactividad, *hooks* de estado/efecto, etc.).
-   **Rutas:** Utiliza las rutas dinámicas del App Router cuando sea necesario, siguiendo la convención de `[slug]`.
-   **APIs:** Las rutas de API deben estar en `app/api/` y seguir el formato de *Route Handlers* (`route.js`).
-   **Análisis:** Al analizar archivos, prioriza la verificación de la correcta utilización de las directivas `'use client'` y la separación entre componentes de servidor y cliente.

# Contexto de la Base de Datos

## Tecnología
-   **Proveedor:** PostgreSQL.
-   **ORM:** Prisma.

## Directrices de Consulta y ORM
-   **Siempre** genera código utilizando el **Prisma Client**. No generes consultas SQL crudas a menos que se solicite explícitamente.
-   **Modelos Clave:**
    -   **`User`**: Representa a un usuario. La columna `role` es un `Enum` (`ADMIN`, `EDITOR`, `READER`).
    -   **`Medicines`**: Contiene información de inventario. La relación con `Category` es uno a muchos.
-   **Requisitos Especiales:** Las consultas que involucren el modelo `Medicines` deben incluir la paginación y el ordenamiento por nombre del creación del medicamento, a menos que se indique lo contrario.

## Relaciones Importantes
-   **Identificar Relaciones:** Identifica las relaciones entre los modelos en base a la estructura de la base de datos descrita por el ORM.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EliNiperd)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EliNiperd)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
