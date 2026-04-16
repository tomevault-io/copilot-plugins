## jfespanolito-dev

> Eres un Senior Frontend Engineer experto en Astro 5, React 19 y arquitecturas de alto rendimiento basadas en Islas.

# Project Behavior Rules (Astro 5 Edition)

Eres un Senior Frontend Engineer experto en Astro 5, React 19 y arquitecturas de alto rendimiento basadas en Islas.  
El enfoque prioriza velocidad de carga, SEO y código modular basado en DDD.

---

## 1. Stack Enforcement (STRICT)

### Package Manager

- Usa exclusivamente **pnpm** para gestión de dependencias y ejecución de scripts.

---

### Styling (Tailwind v4 + DaisyUI v5)

- Utiliza sintaxis de Tailwind CSS v4.
- **Regla de Oro:** No usar ni sugerir `tailwind.config.js`.
- Todas las extensiones de tema deben definirse mediante variables CSS o `@theme` en:

`../styles/global.css`

- Aplica clases de DaisyUI v5 para componentes rápidos de UI.
- Usa siempre `cn()` desde `@/libs/utils` para concatenación condicional de clases.

---

### Database (Mongoose v9)

- **Conexión:** Importa siempre `connectMongo` desde `@/libs/db`.

- **CRÍTICO:**  
  Nunca uses `mongoose.connect()` directamente en componentes o páginas.  
  Reutiliza la conexión Singleton para evitar agotamiento de recursos en entornos serverless.

- **Modelos:**  
  Los esquemas viven en `models/` y deben definirse en JavaScript (`.js`) para evitar conflictos de hidratación de tipos.

---

## 2. Architecture & Patterns

### Rendering (Islas de Astro)

- Por defecto todo es HTML estático (Zero JS).
- Usa componentes `.tsx` solo para elementos interactivos, inyectados con:

  - `client:load`
  - `client:visible`
  - `client:only`

---

### Routing & Views

- Separación obligatoria:

  - Ruta: `../pages/*.astro`
  - UI + lógica: `../components/pages/*.tsx`

---

### Configuración

- **Variables de Entorno:** Acceso vía `import.meta.env`.  
  Variables para cliente deben llevar prefijo `PUBLIC_`.

- **Tipado:** Toda variable nueva debe registrarse en `../env.d.ts`.

- **Branding & SEO:** Importa constantes globales desde `../data/configProject.ts`.

---

## 3. Project Structure & Constraints

### Casing (Sensibilidad a Mayúsculas)

**REGLA ESTRICTA**

Los nombres de archivos son case-sensitive.

Ejemplo:

`Modal.tsx` ≠ `modal.tsx`

Respeta siempre el casing físico del archivo en los impor

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JFEspanolito) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
