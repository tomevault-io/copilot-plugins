## guitar-tuner-pro

> Aplicación web de afinador de guitarra profesional usando React 18, TypeScript, Vite y Tailwind CSS. Sin backend ni base de datos.

# AGENTS.md - Guitar Tuner Pro

## Project Overview
Aplicación web de afinador de guitarra profesional usando React 18, TypeScript, Vite y Tailwind CSS. Sin backend ni base de datos.

## Dependencies

### Production
- **React 19.2.4** - UI framework
- **React Router DOM 7.14.1** - Routing (optional, for multi-page setup)

### Development
- **TypeScript 6.0** - Static typing
- **Vite 8.0** - Build tool and dev server
- **Tailwind CSS 3.4** - Utility-first CSS framework
- **ESLint 9.39** - Code linting

### Native Browser APIs (no npm packages needed)
- `MediaDevices.getUserMedia()` - Microphone access
- `AudioContext` - Audio processing
- `AnalyserNode` - Frequency analysis
- `requestAnimationFrame` - Smooth 60fps UI updates

### No external audio libraries
Implement pitch detection manually using YIN algorithm or autocorrelation

## Coding Standards
- usa camelCase para variables, hooks, funciones
- usa PascalCase para nombre de archivos o nombre de componentes
- usa SNAKE_CASE para constantes
- Usa tailwind 3x para estilos
- No uses inline styles
- Manten principios SOLID
- Utiliza conceptos flexbox para el acomodo de componentes usando tailwind o css
- Manten el nombre de todo en ingles

### TypeScript Rules
- Usar `interface` para objetos y `type` para uniones/primitivas
- Modo estricto siempre activado (`strict: true`)
- Nunca usar `any` - preferir `unknown` o tipos específicos
- Tipar explícitamente funciones que retornan Promise


### Project reomendations
- Manten en consideracion que cada componente o pagina debe de ser responsivo
- No hagas import React si no es necesario
- Manten la paleta de colores y el estilo del layout

### Project src structure
src/
├── components/ # Componentes React reutilizables
├── pages/ # pagitas para rutear
├── hooks/ # Custom hooks (useMicrophone, useAudioTuner)
├── lib/ # Lógica pura de JavaScript/TypeScript
├── types/ # Definiciones de tipos TypeScript
├── utils/ # Funciones helper
├── App.tsx # Componente principal
├── main.tsx # Punto de entrada
└── index.css # Estilos globales (Tailwind)

### TypeScript Rules
- Usar `interface` para objetos y `type` para uniones/primitivas
- Modo estricto siempre activado (`strict: true`)
- Nunca usar `any` - preferir `unknown` o tipos específicos
- Tipar explícitamente funciones que retornan Promise

### Component Patterns
- Componentes funcionales con props tipados
- Exportación nombrada (no default) para mejor tree-shaking
- Props interface con nombre `[ComponentName]Props`

Example:
```tsx
interface ButtonProps {
  variant: 'primary' | 'secondary';
  onClick: () => void;
  children: React.ReactNode;
}

export const Button = ({ variant, onClick, children }: ButtonProps) => {
  // ...
};

---
> Source: [beat-ARDA/guitar-tuner-pro](https://github.com/beat-ARDA/guitar-tuner-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
