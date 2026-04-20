## frontium-videos

> En el proyecto Frontium Videos que utiliza pnpm como package manager exclusivo, es **OBLIGATORIO** usar el comando `pnpm dlx` para todas las operaciones con ShadCN/UI, manteniendo un entorno 100% pnpm.

# Regla ShadCN/UI para Windsurf - Frontium Videos

## Uso de pnpm dlx para ShadCN/UI (OBLIGATORIO)

### Descripción
En el proyecto Frontium Videos que utiliza pnpm como package manager exclusivo, es **OBLIGATORIO** usar el comando `pnpm dlx` para todas las operaciones con ShadCN/UI, manteniendo un entorno 100% pnpm.

### Comando Correcto

#### ✅ CORRECTO
```bash
# Instalación de componentes
pnpm dlx shadcn@latest add button
pnpm dlx shadcn@latest add input
pnpm dlx shadcn@latest add dialog
pnpm dlx shadcn@latest add form
pnpm dlx shadcn@latest add card
pnpm dlx shadcn@latest add table
pnpm dlx shadcn@latest add badge
pnpm dlx shadcn@latest add progress

# Inicialización del proyecto
pnpm dlx shadcn@latest init

# Actualización de componentes
pnpm dlx shadcn@latest update
```

#### ❌ PROHIBIDO
```bash
# NO USAR npm
npx shadcn@latest add button

# NO USAR bun
bunx --bun shadcn@latest add button
bunx shadcn@latest add button

# NO USAR yarn
yarn dlx shadcn@latest add button
```

## Motivación

1. **Entorno 100% pnpm**: Mantiene consistencia total en el uso de pnpm como package manager
2. **Optimización de rendimiento**: Aprovecha las optimizaciones específicas de pnpm para máxima velocidad y eficiencia de espacio
3. **Evita conflictos**: Previene problemas de versiones y dependencias entre diferentes package managers
4. **Coherencia del proyecto**: Mantiene la política estricta de usar únicamente pnpm en Frontium Videos
5. **Compatibilidad**: Asegura que todos los componentes se instalen correctamente con las dependencias de pnpm
6. **Workspace support**: pnpm ofrece mejor soporte para monorepos y workspaces

## Aplicación Obligatoria

Esta regla se aplica a **TODAS** las operaciones con ShadCN/UI:

- **Instalación inicial** de componentes
- **Añadir nuevos componentes** durante el desarrollo
- **Actualización** de componentes existentes
- **Configuración inicial** del proyecto ShadCN/UI
- **Reinstalación** de componentes

## Configuración Inicial del Proyecto

Para configurar ShadCN/UI por primera vez en Frontium Videos:

```bash
# 1. Inicializar ShadCN/UI
pnpm dlx shadcn@latest init

# 2. Añadir componentes básicos para la plataforma de cursos
pnpm dlx shadcn@latest add button
pnpm dlx shadcn@latest add input
pnpm dlx shadcn@latest add card
pnpm dlx shadcn@latest add dialog
pnpm dlx shadcn@latest add form
pnpm dlx shadcn@latest add table
pnpm dlx shadcn@latest add tabs
pnpm dlx shadcn@latest add avatar
pnpm dlx shadcn@latest add badge
pnpm dlx shadcn@latest add progress
```

## Estructura de Componentes

Los componentes ShadCN/UI se instalan en:

```
src/
└── components/
    └── ui/              # Componentes ShadCN/UI
        ├── button.tsx
        ├── input.tsx
        ├── card.tsx
        └── ...
```

## Importación de Componentes

Seguir el patrón de importación con alias:

```tsx
// ✅ CORRECTO
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Card, CardContent, CardHeader } from '@/components/ui/card';

// ❌ INCORRECTO
import { Button } from '../components/ui/button';
import { Input } from './ui/input';
```

---

**IMPORTANTE**: Esta regla es **obligatoria** para mantener la consistencia del entorno de desarrollo en Frontium Videos. Cualquier instalación de componentes ShadCN/UI que no siga esta regla debe ser corregida.

**Aplica a todos los asistentes de IA**: Cursor AI, GitHub Copilot, Windsurf y cualquier otro AI que opere sobre este repositorio.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JordiNodeJS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
