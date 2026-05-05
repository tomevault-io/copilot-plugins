## styling

> Padrões de estilização com Tailwind CSS e design system


# Padrões de Estilização

## Stack

- **Tailwind CSS**: v4
- **shadcn/ui**: New York style
- **Ícones**: Lucide React
- **Animações**: Framer Motion + GSAP

## Configuração

### Tailwind CSS
- **Config**: `postcss.config.mjs`
- **CSS Global**: `src/app/globals.css`
- **Theme**: Dark mode por padrão

### shadcn/ui
- **Config**: `components.json`
- **Style**: New York
- **Base Color**: Neutral
- **CSS Variables**: Habilitado

## Design System

### Cores Principais

```css
/* Background */
--background: 0 0% 98%        /* Light: quase branco */
--background: 0 0% 4%          /* Dark: #0a0a0f */

/* Primary (verde/amarelo) */
--primary: 84 76% 55%
--primary-foreground: 0 0% 15%

/* Cards */
--card: 0 0% 100%              /* Light */
--card: 0 0% 18%              /* Dark: #1a1a2e */
```

### Cores Customizadas

```typescript
// Background principal
bg-[#0a0a0f]

// Glassmorphism
bg-[#0a0a0f]/80 backdrop-blur-xl

// Borders sutis
border-white/10

// Primary color
text-primary
bg-primary
```

## Padrões Visuais

### Glassmorphism

```typescript
// Header e cards principais
className="bg-[#0a0a0f]/80 backdrop-blur-xl border-b border-white/10"
```

### Containers

```typescript
// Container padrão (max-width + padding)
className="max-w-6xl mx-auto px-4"

// Main content (compensa header fixo)
className="max-w-6xl mx-auto px-4 pt-24 pb-8"
```

### Spacing

```typescript
// Gaps comuns
gap-6        // Entre elementos principais
gap-4        // Entre elementos secundários
gap-2        // Entre elementos pequenos

// Padding
p-4          // Padrão
px-4         // Horizontal
py-5         // Vertical (header)
pt-24        // Top (compensa header fixo)
pb-8         // Bottom
```

### Borders

```typescript
// Bordas sutis
border border-white/10
border-b border-white/10

// Radius
rounded-xl   // Cards principais
rounded-lg   // Botões, inputs
```

## Componentes UI (shadcn/ui)

### Variantes de Botão

```typescript
<Button variant="default">Padrão</Button>
<Button variant="secondary">Secundário</Button>
<Button variant="outline">Outline</Button>
<Button variant="ghost">Ghost</Button>
<Button variant="destructive">Destrutivo</Button>
```

### Cards

```typescript
<Card>
  <CardHeader>
    <CardTitle>Título</CardTitle>
    <CardDescription>Descrição</CardDescription>
  </CardHeader>
  <CardContent>
    Conteúdo
  </CardContent>
</Card>
```

### Inputs

```typescript
<Input placeholder="Digite aqui..." />
<Textarea placeholder="Mensagem..." />
<Label>Nome</Label>
```

## Animações

### Transições

```typescript
// Transições suaves
transition-all
transition-colors
transition-opacity

// Duração
duration-200
duration-300
```

### Hover Effects

```typescript
// Glow effect
group-hover:opacity-100
group-hover:from-primary/30
group-hover:to-primary/10

// Text color
group-hover:text-primary
```

### Framer Motion

```typescript
import { motion } from "framer-motion"

<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.3 }}
>
  Conteúdo animado
</motion.div>
```

## Utilitário `cn()`

### Uso

```typescript
import { cn } from "@/lib/utils"

// Merge classes
<div className={cn(
  "base-class",
  condition && "conditional-class",
  anotherCondition ? "class-a" : "class-b"
)} />
```

### Implementação

```typescript
// src/lib/utils.ts
import { clsx, type ClassValue } from "clsx"
import { twMerge } from "tailwind-merge"

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

## Responsividade

### Breakpoints Tailwind

```typescript
// Mobile first
className="text-sm md:text-base lg:text-lg"

// Grid responsivo
className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3"
```

### Hook Mobile

```typescript
import { useMobile } from "@/hooks/use-mobile"

function Component() {
  const isMobile = useMobile()
  
  return (
    <div className={isMobile ? "flex-col" : "flex-row"}>
      {/* ... */}
    </div>
  )
}
```

## Ícones (Lucide React)

### Uso

```typescript
import { MessageSquare, Library, Calendar } from "lucide-react"

<MessageSquare className="h-5 w-5" />
<Library className="h-6 w-6 text-primary" />
```

### Tamanhos Comuns

- `h-4 w-4` - Pequeno (16px)
- `h-5 w-5` - Médio (20px)
- `h-6 w-6` - Grande (24px)

### Props Úteis

```typescript
<Icon
  className="h-5 w-5"
  strokeWidth={2.5}  // Espessura da linha
  color="currentColor" // Herda cor do texto
/>
```

## Dark Mode

### Configuração

- Dark mode é o padrão (`className="dark"` no `<html>`)
- Cores definidas em `globals.css` com variáveis CSS
- shadcn/ui gerencia automaticamente

### Forçar Modo

```typescript
// Em src/app/layout.tsx
<html lang="pt-BR" className="dark" suppressHydrationWarning>
```

## Acessibilidade

### Contraste

- Texto em `foreground` vs `background`
- Botões com `primary-foreground` em `primary`
- Verificar contraste mínimo (WCAG AA)

### Focus States

```typescript
// Focus visível
focus:outline-none focus:ring-2 focus:ring-primary
```

## Performance

### Otimizações

- Usar classes Tailwind (não CSS customizado quando possível)
- Evitar `@apply` excessivo
- Usar `backdrop-blur-xl` com moderação (pode ser pesado)

### Critical CSS

- Tailwind purga automaticamente classes não usadas
- CSS é gerado apenas com classes utilizadas

---
> Source: [zoryon-dev/maquina-de-conteudo](https://github.com/zoryon-dev/maquina-de-conteudo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
