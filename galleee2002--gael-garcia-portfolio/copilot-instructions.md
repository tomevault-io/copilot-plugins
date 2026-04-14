## gael-garcia-portfolio

> Estas reglas DEBEN ser leídas y seguidas ANTES de ejecutar cualquier prompt o tarea en este proyecto.

# 🎯 Reglas de Cursor AI para Gael García Portfolio

## 📋 INSTRUCCIONES GENERALES

Estas reglas DEBEN ser leídas y seguidas ANTES de ejecutar cualquier prompt o tarea en este proyecto.

---

## 🏗️ ARQUITECTURA DEL PROYECTO

### Patrón Atomic Design

Este proyecto sigue ESTRICTAMENTE el patrón Atomic Design:

```
Atoms → Molecules → Organisms → Templates → Pages
```

**NUNCA:**

- Mezclar niveles jerárquicos incorrectamente
- Crear un componente en el nivel incorrecto
- Importar un nivel superior en un nivel inferior (ej: Organism en Atom)

**SIEMPRE:**

- Atoms solo contienen componentes UI básicos sin dependencias
- Molecules combinan Atoms
- Organisms combinan Molecules y Atoms
- Templates definen layouts sin contenido específico
- Pages son instancias de Templates con contenido real

---

## 📁 ESTRUCTURA DE ARCHIVOS

### Organización de Componentes

Cada componente DEBE seguir esta estructura EXACTA:

```
ComponentName/
├── ComponentName.tsx    (PascalCase, mismo nombre que la carpeta)
└── index.ts            (export default únicamente)
```

**PROHIBIDO:**

- Crear múltiples componentes en un solo archivo
- Usar nombres diferentes entre carpeta y archivo
- Exportar múltiples componentes desde index.ts
- Crear archivos de estilos CSS separados (usar Tailwind)

---

## 💻 CONVENCIONES DE CÓDIGO

### 1. Nomenclatura

**Componentes (Archivos .tsx):**

- SIEMPRE PascalCase: `Button.tsx`, `FormField.tsx`, `ProjectCard.tsx`
- El nombre del archivo DEBE coincidir con el nombre del componente
- El nombre de la carpeta DEBE coincidir con el nombre del componente

**Hooks:**

- SIEMPRE prefijo `use` + camelCase: `useWindowSize.ts`, `useAuth.ts`
- Un hook por archivo

**Utilidades:**

- SIEMPRE camelCase: `formatters.ts`, `validators.ts`
- Agrupar funciones relacionadas

**Tipos:**

- SIEMPRE PascalCase con sufijo `.types.ts`: `common.types.ts`, `user.types.ts`

### 2. Imports

**OBLIGATORIO usar alias de importación:**

```typescript
// ✅ CORRECTO
import Button from "@atoms/Button";
import FormField from "@molecules/FormField";
import Navbar from "@organisms/Navbar";
import MainLayout from "@templates/MainLayout";
import Home from "@pages/Home";
import { useWindowSize } from "@hooks/useWindowSize";
import { formatDate } from "@utils/formatters";
import type { Project } from "@types/common.types";

// ❌ INCORRECTO
import Button from "../../components/atoms/Button";
import FormField from "../molecules/FormField";
```

**Orden de imports (SIEMPRE en este orden):**

1. React y librerías externas
2. Componentes (atoms → molecules → organisms → templates → pages)
3. Hooks
4. Utils
5. Types
6. Estilos/Assets

### 3. Sintaxis TypeScript

**SIEMPRE:**

- Usar TypeScript strict mode
- Definir interfaces para props: `ComponentNameProps`
- Tipar todas las funciones, variables y parámetros
- Usar `React.FC<PropsType>` para componentes funcionales
- Preferir `interface` sobre `type` para objetos
- Usar `type` para unions, intersections y primitivas

**Ejemplo de componente correcto:**

```typescript
import React from "react";

interface ButtonProps {
  children: React.ReactNode;
  variant?: "primary" | "secondary" | "outline";
  size?: "sm" | "md" | "lg";
  onClick?: () => void;
  disabled?: boolean;
  className?: string;
}

const Button: React.FC<ButtonProps> = ({
  children,
  variant = "primary",
  size = "md",
  onClick,
  disabled = false,
  className = "",
}) => {
  // lógica del componente

  return <button>{children}</button>;
};

export default Button;
```

### 4. Estilos

**SIEMPRE usar Tailwind CSS:**

- NO crear archivos CSS separados para componentes
- Usar clases de Tailwind directamente en JSX
- Para estilos dinámicos, usar template literals
- Mantener consistencia con las variables CSS en `src/style.css`

**PROHIBIDO:**

- Inline styles con el atributo `style`
- Importar archivos CSS en componentes (excepto main.tsx)
- Usar CSS modules

### 5. Formato de Código

**Prettier/Formatter configuración:**

- Comillas dobles `"` para strings (NO comillas simples)
- Punto y coma `;` al final de statements
- Trailing commas en objetos y arrays
- 2 espacios de indentación
- Max line length: 80-100 caracteres
- Arrow functions con paréntesis: `(x) => x`

---

## 🚫 PROHIBICIONES ESTRICTAS

### NUNCA hacer lo siguiente:

1. **Duplicar código:**

   - Si un código se repite 2+ veces, crear un componente/función reutilizable
   - Revisar si ya existe un componente similar antes de crear uno nuevo

2. **Modificar archivos de configuración sin razón:**
   - `tsconfig.json`
   - `vite.config.ts`
   - `package.json` (scripts)
3. **Crear componentes fuera de la estructura Atomic Design:**

   - TODO componente debe estar en atoms/molecules/organisms/templates/pages
   - NO crear carpeta "components/common" o similar

4. **Ignorar TypeScript errors:**

   - TODO código debe compilar sin errores de TypeScript
   - NO usar `@ts-ignore` o `any` sin justificación

5. **Desestructurar incorrectamente:**

   - Siempre desestructurar props en la firma de la función
   - NO acceder a props con `props.nombreProp` dentro del componente

6. **Crear archivos innecesarios:**
   - NO crear archivos de test (a menos que se solicite explícitamente)
   - NO crear archivos de configuración adicionales
   - NO crear archivos .spec.ts o .test.ts automáticamente

---

## ✅ MEJORES PRÁCTICAS OBLIGATORIAS

### 1. Componentes

**Cada componente DEBE:**

- Tener una sola responsabilidad (Single Responsibility Principle)
- Ser reutilizable cuando sea posible
- Tener props bien tipadas con TypeScript
- Incluir valores por defecto para props opcionales
- Usar destructuring en la firma de la función

**Tamaño de componentes:**

- Atoms: 20-50 líneas máximo
- Molecules: 50-100 líneas máximo
- Organisms: 100-200 líneas máximo
- Si excede estos límites, considerar dividir el componente

### 2. Estado y Props

**Props:**

- SIEMPRE tipar con interface `ComponentNameProps`
- Colocar la interface ANTES del componente
- Usar `?` para props opcionales
- Documentar props complejas con JSDoc comments

**Estado:**

- Usar `useState` para estado local simple
- Colocar `useState` al inicio del componente
- Nombrar estado con pares: `[value, setValue]` o `[isOpen, setIsOpen]`

### 3. Funciones y Eventos

**Handlers de eventos:**

- Nombrar con prefijo `handle`: `handleClick`, `handleSubmit`, `handleChange`
- Colocar handlers ANTES del return
- Tipar parámetros de eventos: `React.MouseEvent`, `React.ChangeEvent<HTMLInputElement>`

**Funciones auxiliares:**

- Colocar DENTRO del componente si usan props/state
- Colocar en `/utils` si son puras y reutilizables

### 4. Comentarios

**SIEMPRE comentar:**

- Lógica compleja que no es obvia
- Funciones de utilidad con JSDoc
- Tipos complejos con descripción

**NUNCA comentar:**

- Código obvio
- Código que se explica por sí mismo
- TODO lo que sea autoexplicativo

### 5. Performance

**SIEMPRE:**

- Usar `React.memo` para componentes que se renderizan frecuentemente
- Usar `useCallback` para funciones pasadas como props
- Usar `useMemo` para cálculos costosos
- Evitar crear funciones/objetos dentro del JSX

---

## 📝 PROCESO ANTES DE CREAR/MODIFICAR CÓDIGO

### Checklist OBLIGATORIO antes de crear un componente:

1. ✅ ¿Ya existe un componente similar?
2. ✅ ¿Cuál es el nivel correcto en Atomic Design?
3. ✅ ¿El nombre sigue la convención PascalCase?
4. ✅ ¿Las props están correctamente tipadas?
5. ✅ ¿Usa los alias de importación?
6. ✅ ¿Sigue el formato de carpeta/archivo establecido?
7. ✅ ¿El código está formateado correctamente?
8. ✅ ¿No hay duplicación de código?

### Checklist OBLIGATORIO antes de modificar código existente:

1. ✅ ¿He leído el código existente completo?
2. ✅ ¿Entiendo la estructura actual?
3. ✅ ¿Mi cambio respeta la arquitectura?
4. ✅ ¿No rompo funcionalidad existente?
5. ✅ ¿Mantengo el estilo de código consistente?
6. ✅ ¿No introduzco duplicación?

---

## 🔄 FLUJO DE TRABAJO

### Al recibir una solicitud de nuevo componente:

1. **Identificar el nivel de Atomic Design**

   - ¿Es un elemento UI básico? → Atom
   - ¿Combina 2-3 átomos? → Molecule
   - ¿Es una sección completa? → Organism
   - ¿Es un layout? → Template
   - ¿Es una página con contenido? → Page

2. **Verificar componentes existentes**

   - Buscar en la carpeta correspondiente
   - Revisar si se puede reutilizar o extender uno existente

3. **Crear estructura de archivos**

   - Crear carpeta con nombre del componente
   - Crear ComponentName.tsx
   - Crear index.ts

4. **Implementar el componente**

   - Definir interface de props
   - Implementar componente funcional
   - Añadir estilos con Tailwind
   - Export default

5. **Verificar**
   - Compilación sin errores
   - Formato de código correcto
   - Imports usando alias
   - Sin código duplicado

### Al recibir una solicitud de modificación:

1. **Leer el archivo completo**
2. **Entender el contexto y dependencias**
3. **Hacer cambios mínimos necesarios**
4. **Mantener el estilo existente**
5. **Verificar que no se rompe nada**

---

## 🎨 ESTILOS CON TAILWIND

### Reglas para usar Tailwind:

**SIEMPRE:**

- Usar utility classes de Tailwind
- Agrupar clases relacionadas: layout → spacing → sizing → colors → effects
- Usar variables CSS de `src/style.css` cuando sea apropiado
- Mantener consistencia de colores:
  - Primary: `blue-600`
  - Secondary: `purple-600`
  - Accent: `amber-500`
  - Gray: `gray-*`

**Template para clases dinámicas:**

```typescript
const baseStyles = "clase1 clase2 clase3";
const variantStyles = {
  primary: "bg-blue-600 text-white",
  secondary: "bg-purple-600 text-white",
};

<div className={`${baseStyles} ${variantStyles[variant]}`}>
```

---

## 📦 GESTIÓN DE DEPENDENCIAS

### Antes de instalar una dependencia:

1. ✅ ¿Es realmente necesaria?
2. ✅ ¿No se puede implementar con las herramientas existentes?
3. ✅ ¿Está bien mantenida? (última actualización < 6 meses)
4. ✅ ¿No tiene vulnerabilidades conocidas?
5. ✅ ¿Es la más ligera disponible?

### PROHIBIDO instalar:

- Librerías de UI completas (Material-UI, Ant Design, etc.) - Ya tenemos Tailwind
- jQuery o similares - Usamos React
- Librerías obsoletas o sin mantenimiento
- Múltiples librerías que hacen lo mismo

---

## 🐛 MANEJO DE ERRORES

### SIEMPRE:

- Validar inputs de usuario
- Manejar casos edge (null, undefined, arrays vacíos)
- Proporcionar feedback visual apropiado
- Usar try-catch para operaciones asíncronas

### NUNCA:

- Dejar errores sin manejar
- Usar `console.log` en producción (usar solo para debug temporal)
- Mostrar errores técnicos al usuario
- Ignorar warnings de TypeScript

---

## 📚 DOCUMENTACIÓN

### Cuándo actualizar documentación:

**SIEMPRE actualizar cuando:**

- Creas un nuevo nivel de componente (nuevo tipo de Atom, Molecule, etc.)
- Añades una nueva carpeta en `/src`
- Cambias la estructura del proyecto
- Añades nuevas convenciones o reglas

**Archivos a actualizar:**

- README específico de la carpeta afectada
- ARCHITECTURE.md si afecta la arquitectura
- PROJECT_STRUCTURE.md si afecta la estructura

### NO actualizar documentación para:

- Componentes individuales rutinarios
- Cambios menores de estilos
- Fixes de bugs sin impacto arquitectónico

---

## 🚀 ANTES DE COMPLETAR UNA TAREA

### Checklist final OBLIGATORIO:

1. ✅ El código compila sin errores de TypeScript
2. ✅ No hay warnings de linter
3. ✅ El formato es consistente (comillas dobles, punto y coma, etc.)
4. ✅ Los imports usan alias correctamente
5. ✅ No hay código duplicado
6. ✅ Los nombres siguen las convenciones
7. ✅ La estructura de archivos es correcta
8. ✅ Se respeta Atomic Design
9. ✅ El código es limpio y legible
10. ✅ No se crearon archivos innecesarios

---

## ⚠️ CASOS ESPECIALES

### Si el usuario pide algo que rompe estas reglas:

1. **Informar al usuario** sobre la regla que se rompería
2. **Sugerir alternativa** que respete las reglas
3. **Solo si insiste**, hacer lo solicitado pero documentar la excepción

### Si encuentras código existente que no sigue estas reglas:

1. **NO cambiar** todo el código inmediatamente
2. **Aplicar las reglas** solo al código nuevo o modificado
3. **Mantener consistencia** con el código circundante
4. **Sugerir refactor** si es apropiado

---

## 📊 PRIORIDADES

En orden de importancia:

1. **Funcionalidad correcta** - El código debe funcionar
2. **TypeScript sin errores** - Debe compilar
3. **Arquitectura Atomic Design** - Respetar la estructura
4. **Convenciones de código** - Formato y estilo
5. **Performance** - Optimización cuando sea necesaria
6. **Documentación** - Mantener docs actualizados

---

## 🎯 RESUMEN EJECUTIVO

**Antes de CUALQUIER acción:**

1. ✅ Leer estas reglas
2. ✅ Verificar que entiendes la tarea
3. ✅ Planificar respetando Atomic Design
4. ✅ Verificar que no duplicas código
5. ✅ Usar alias de importación
6. ✅ Mantener formato consistente
7. ✅ Verificar antes de completar

**Recuerda:**

- 🏗️ Atomic Design es OBLIGATORIO
- 📁 Estructura de carpetas es SAGRADA
- 💻 TypeScript strict es REQUERIDO
- 🎨 Tailwind CSS es el ESTÁNDAR
- 📝 Formato consistente es ESENCIAL
- 🚫 Duplicación de código está PROHIBIDA

---

**Última actualización:** Octubre 2025
**Versión:** 1.0.0

---

## 🤖 PARA CURSOR AI

Este archivo DEBE ser consultado ANTES de:

- Crear cualquier archivo nuevo
- Modificar código existente
- Instalar dependencias
- Cambiar estructura de proyecto
- Responder a cualquier prompt del usuario

**NO proceder con una tarea si:**

- Rompe las reglas de Atomic Design
- Duplica código existente
- No sigue las convenciones de nomenclatura
- Introduce errores de TypeScript
- Crea archivos fuera de la estructura establecida

**SIEMPRE pregunta al usuario si:**

- La tarea requiere romper alguna regla
- No está claro el nivel de Atomic Design apropiado
- Se necesita instalar una nueva dependencia
- El cambio afecta múltiples archivos

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Galleee2002) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
