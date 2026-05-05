## components

> Padrões e convenções para componentes React


# Padrões de Componentes

## Estrutura de Componentes

### Localização
- **UI Base**: `src/components/ui/` - Componentes shadcn/ui
- **Feature**: `src/components/[feature]/` - Componentes específicos
- **Layout**: `src/components/app-layout.tsx` - Layout principal
- **Auth**: `src/components/auth/` - Componentes de autenticação

## Server vs Client Components

### Regra Geral
**Padrão**: Server Components (sem `"use client"`)

### Quando Usar Client Component
Use `"use client"` APENAS quando necessário:
- ✅ Hooks do React (`useState`, `useEffect`, `useMemo`, etc)
- ✅ Event handlers (`onClick`, `onChange`, etc)
- ✅ Browser APIs (`window`, `document`, `localStorage`, etc)
- ✅ Context API (`createContext`, `useContext`)
- ✅ Bibliotecas que requerem client (`framer-motion`, `gsap`)

### Quando NÃO Usar Client Component
- ❌ Apenas para exibir dados
- ❌ Apenas para estilização
- ❌ Quando pode ser Server Component

### Exemplo: Isolamento de Client Components

```typescript
// ✅ BOM: Server Component (padrão)
export function UserProfile({ userId }: { userId: string }) {
  const user = await getUser(userId) // Server Component pode fazer fetch
  
  return (
    <div>
      <h1>{user.name}</h1>
      <InteractiveButton /> {/* Client Component isolado */}
    </div>
  )
}

// ✅ BOM: Client Component isolado e pequeno
"use client"
export function InteractiveButton() {
  const [count, setCount] = useState(0)
  return <button onClick={() => setCount(count + 1)}>{count}</button>
}
```

## Componente: AppLayout

**Arquivo**: `src/components/app-layout.tsx`

### Estrutura
- **Tipo**: Client Component (usa hooks e animações)
- **Props**: `{ children, className? }`
- **Responsabilidades**:
  - Header fixo com glassmorphism
  - Navbar animada (tubelight)
  - Layout de conteúdo principal

### Padrões Visuais
- Header: `fixed top-0` com `z-40`
- Glassmorphism: `bg-[#0a0a0f]/80 backdrop-blur-xl`
- Padding vertical no header: `pt-5 pb-5`
- Main content: `pt-24` (compensa header fixo)

### NavItems
```typescript
const navItems = [
  { name: "Chat", url: "/dashboard", icon: MessageSquare },
  { name: "Biblioteca", url: "/library", icon: Library },
  { name: "Calendário", url: "/calendar", icon: Calendar },
  { name: "Fontes", url: "/sources", icon: Globe },
  { name: "Configurações", url: "/settings", icon: Settings },
]
```

## Componentes de Autenticação

### SignInCard / SignUpCard
- **Tipo**: Client Component
- **Localização**: `src/components/auth/sign-in-card.tsx`
- **Uso**: Páginas de autenticação customizadas

### UserMenu
- **Tipo**: Client Component
- **Localização**: `src/components/auth/user-menu.tsx`
- **Uso**: Menu dropdown do usuário no header
- **Componente**: `<UserButton />` do Clerk

### OAuthButtons
- **Tipo**: Client Component
- **Localização**: `src/components/auth/oauth-buttons.tsx`
- **Uso**: Botões de OAuth (Google, GitHub)

## Componentes UI (shadcn/ui)

### Padrão de Uso
- **Import**: `import { Button } from "@/components/ui/button"`
- **Estilo**: New York style (configurado em `components.json`)
- **Customização**: Via props e className

### Componentes Disponíveis
- `button`, `card`, `input`, `textarea`, `label`
- `dialog`, `dropdown-menu`, `tooltip`
- `badge`, `alert`, `skeleton`, `spinner`
- `separator`, `progress`, `switch`
- `sidebar`, `menubar`, `radio-group`

### Exemplo de Uso
```typescript
import { Button } from "@/components/ui/button"
import { Card } from "@/components/ui/card"

export function MyComponent() {
  return (
    <Card>
      <Button variant="default">Clique aqui</Button>
    </Card>
  )
}
```

## Componentes de Feature

### AnimatedAIChat
- **Localização**: `src/components/dashboard/animated-ai-chat.tsx`
- **Tipo**: Client Component (animações)
- **Uso**: Interface de chat com IA

### ModelSelector
- **Localização**: `src/components/chat/model-selector.tsx`
- **Tipo**: Client Component (interatividade)
- **Uso**: Seletor de modelos de IA

## Padrões de Estilização

### Tailwind CSS
- **Config**: Tailwind CSS 4
- **Customização**: `src/app/globals.css`
- **Tema**: Dark mode por padrão

### Classes Comuns
- **Container**: `max-w-6xl mx-auto px-4`
- **Glassmorphism**: `bg-[#0a0a0f]/80 backdrop-blur-xl`
- **Borders**: `border border-white/10`
- **Spacing**: `gap-6`, `p-4`, `pt-24`

### Utilitário `cn()`
```typescript
import { cn } from "@/lib/utils"

// Merge classes com tailwind-merge
<div className={cn("base-class", condition && "conditional-class")} />
```

## Props e TypeScript

### Padrão de Props
```typescript
// ✅ BOM: Interface explícita
interface ComponentProps {
  title: string
  description?: string
  children: React.ReactNode
}

export function Component({ title, description, children }: ComponentProps) {
  // ...
}

// ❌ EVITAR: Props inline sem tipo
export function Component(props: any) {
  // ...
}
```

### Props com Children
```typescript
interface LayoutProps {
  children: React.ReactNode
  className?: string
}
```

## Hooks Customizados

### Localização
- `src/hooks/use-mobile.ts` - Detecção de mobile

### Padrão
```typescript
// Nome: use[Feature]
export function useMobile() {
  // ...
}
```

## Performance

### Otimizações
- **React.memo()**: Para componentes que re-renderizam frequentemente
- **useMemo()**: Para cálculos caros
- **useCallback()**: Para funções passadas como props

### Lazy Loading
```typescript
import dynamic from "next/dynamic"

const HeavyComponent = dynamic(() => import("./heavy-component"), {
  loading: () => <Skeleton />,
})
```

---
> Source: [zoryon-dev/maquina-de-conteudo](https://github.com/zoryon-dev/maquina-de-conteudo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
