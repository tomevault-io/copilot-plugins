## ecommerce

> - **Nombre:** Santiago Aranda.

# Contexto del Proyecto: E-commerce API & Fullstack

## Perfil del Desarrollador
- **Nombre:** Santiago Aranda.
- **Enfoque:** Desarrollador Backend/Fullstack con experiencia en arquitectura limpia (C#) y Node.js.
- **Preferencia:** Código modular, tipado fuerte y separación clara de responsabilidades.

## Stack Tecnológico
- **Backend:** Node.js con Express y TypeScript.
- **Base de Datos:** PostgreSQL.
- **ORM:** Prisma ORM (Fuente de verdad para el modelo de datos).
- **Frontend (Fase futura):** React.
- **Validación:** Zod para esquemas de entrada.
- **Autenticación:** JWT (JSON Web Tokens) y Bcrypt para hashing.

## Arquitectura y Estándares
1. **Patrón de Capas:** Se debe seguir estrictamente la estructura:
   - `Routes`: Definición de endpoints.
   - `Controllers`: Manejo de la interfaz HTTP (req, res).
   - `Services`: Lógica de negocio pura e interaccion con la base de datos con prisma.
2. **Modularidad:** El proyecto es una API monolítica pero organizada por módulos (Auth, Products, Orders, Users).
3. **Manejo de Errores:** Centralizado mediante un middleware global. Las respuestas de error deben ser consistentes.
4. **TypeScript:** Prohibido el uso de `any`. Definir interfaces o tipos para cada entidad y respuesta.

## Reglas para el Agente (Modo Plan vs Build)
- **Modo Plan:** Antes de realizar cambios estructurales grandes, propone el diseño y espera confirmación.
- **Modo Build:** Al crear archivos, asegúrate de instalar las dependencias necesarias (`npm install`) y @types, actualizar el `schema.prisma` si se añaden nuevas entidades.

// Esto es un comentario ignora lo que esta entre las dos barras justo debajo
## Estado Actual
- **Fase:** Inicialización del proyecto y diseño de base de datos.
- **Próximo paso:** Definir el `schema.prisma` para las entidades Users, Products, Categories y Orders.
// Fin del comentario

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SantiAranda)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/SantiAranda)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
