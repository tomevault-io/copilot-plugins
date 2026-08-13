## design-system-guide

> - **Nome do produto:** ZapMonitor


# ZapMonitor — Documento de Padrões de Design

## 1. Identidade da Marca

### Nome e Logotipo

- **Nome do produto:** ZapMonitor
- **Abreviação:** ZM (usada no sidebar colapsado)
- **Tipografia do logo:**
  - `Zap` — texto branco sobre fundo escuro (`#1D293D`), `font-bold`, `text-2xl`
  - `Monitor` — destaque na cor primária (`#00C950` / `rgb(34 197 94)`)
- **Logo SVG:** `public/assets/logo.svg` (versão horizontal, 428×136px)
- **Favicon SVG:** `public/favicon.svg` (versão icônica, 110×110px, `border-radius: 16px`)

### Cores da Marca (Brand Colors)

| Token       | Valor     | Uso                                   |
| ----------- | --------- | ------------------------------------- |
| Brand Green | `#00C950` | Cor primária, destaque do logotipo    |
| Brand Dark  | `#1D293D` | Fundo do logo, cor do texto principal |
| White       | `#FFFFFF` | Elementos sobre fundos escuros        |

---

## 2. Sistema de Cores (Design Tokens — CSS Variables)

### Modo Claro (`:root`)

| Token                    | Valor                                  | Descrição                                   |
| ------------------------ | -------------------------------------- | ------------------------------------------- |
| `--background`           | `oklch(1 0 0)` → branco                | Fundo principal da aplicação                |
| `--foreground`           | `rgb(29 41 61)` → `#1D293D`            | Texto principal                             |
| `--card`                 | `oklch(1 0 0)` → branco                | Fundo de cards                              |
| `--card-foreground`      | `rgb(29 41 61)`                        | Texto em cards                              |
| `--primary`              | `rgb(34 197 94)` → `#22C55E`           | Cor de ação primária (botões, links ativos) |
| `--primary-foreground`   | `oklch(0.982 0.018 155.826)`           | Texto sobre elementos primários             |
| `--secondary`            | `oklch(0.967 0.001 286.375)`           | Fundo secundário / botões secundários       |
| `--secondary-foreground` | `oklch(0.21 0.006 285.885)`            | Texto em elementos secundários              |
| `--muted`                | `oklch(0.967 0.001 286.375)`           | Fundo de elementos silenciados              |
| `--muted-foreground`     | `oklch(0.552 0.016 285.938)`           | Texto desabilitado / descrições             |
| `--accent`               | `oklch(0.967 0.001 286.375)`           | Hover de itens de menu                      |
| `--destructive`          | `oklch(0.577 0.245 27.325)` → vermelho | Ações destrutivas / erros                   |
| `--border`               | `oklch(0.92 0.004 286.32)`             | Bordas de inputs e divisores                |
| `--input`                | `oklch(0.92 0.004 286.32)`             | Cor de borda de inputs                      |
| `--ring`                 | `rgb(34 197 94)`                       | Anel de foco (acessibilidade)               |
| `--radius`               | `0.65rem`                              | Raio de borda padrão                        |

### Modo Escuro (`.dark`)

| Token           | Valor                             | Descrição                          |
| --------------- | --------------------------------- | ---------------------------------- |
| `--background`  | `rgb(29 41 61)` → `#1D293D`       | Fundo escuro principal             |
| `--foreground`  | `oklch(0.985 0 0)` → quase branco | Texto principal                    |
| `--card`        | `oklch(0.21 0.006 285.885)`       | Fundo de cards escuros             |
| `--primary`     | `oklch(0.696 0.17 162.48)`        | Verde levemente mais claro no dark |
| `--secondary`   | `oklch(0.274 0.006 286.033)`      | Fundo secundário escuro            |
| `--muted`       | `oklch(0.274 0.006 286.033)`      | Elementos silenciados no dark      |
| `--destructive` | `oklch(0.704 0.191 22.216)`       | Vermelho mais suave no dark        |
| `--border`      | `oklch(1 0 0 / 10%)`              | Bordas sutis no dark               |
| `--input`       | `oklch(1 0 0 / 15%)`              | Inputs no dark                     |

### Cores dos Gráficos (Chart Colors)

| Token       | Modo Claro                                 | Modo Escuro                            |
| ----------- | ------------------------------------------ | -------------------------------------- |
| `--chart-1` | `oklch(0.646 0.222 41.116)` (laranja)      | `oklch(0.488 0.243 264.376)` (azul)    |
| `--chart-2` | `oklch(0.6 0.118 184.704)` (verde-azulado) | `oklch(0.696 0.17 162.48)` (verde)     |
| `--chart-3` | `oklch(0.398 0.07 227.392)` (azul escuro)  | `oklch(0.769 0.188 70.08)` (amarelo)   |
| `--chart-4` | `oklch(0.828 0.189 84.429)` (amarelo)      | `oklch(0.627 0.265 303.9)` (roxo)      |
| `--chart-5` | `oklch(0.769 0.188 70.08)` (dourado)       | `oklch(0.645 0.246 16.439)` (vermelho) |

### Cores Utilitárias Aplicadas Diretamente (Tailwind)

Algumas páginas usam classes Tailwind hardcoded além dos tokens:

| Classe                                | Uso                                           |
| ------------------------------------- | --------------------------------------------- |
| `bg-green-600` / `hover:bg-green-700` | Botão de atalho WhatsApp na Home              |
| `bg-blue-100` / `text-blue-600`       | Card de mensagens enviadas (Reports)          |
| `bg-green-100` / `text-green-600`     | Card de mensagens recebidas (Reports)         |
| `bg-purple-100` / `text-purple-600`   | Card de atendimento humano (Reports)          |
| `text-gray-600`                       | Textos descritivos nos Reports                |
| `bg-gray-50`                          | Fundo da área de conteúdo principal no layout |
| `bg-yellow-300`                       | Banner de aviso de desconexão                 |
| `#3b82f6` (blue-500)                  | Barra de gráficos "Sent"                      |
| `#10b981` (emerald-500)               | Barra de gráficos "Received"                  |

---

## 3. Tipografia

O projeto **não define uma família tipográfica customizada** — herda os padrões do Tailwind CSS (system font stack). Os tamanhos e pesos aplicados são:

| Uso                               | Classe Tailwind                 | Resultado     |
| --------------------------------- | ------------------------------- | ------------- |
| Título de página (H1 em `Header`) | `text-2xl font-bold`            | 1.5rem, 700   |
| Título de seção na Home           | `text-2xl`                      | 1.5rem, 400   |
| Título do Dashboard               | `text-3xl`                      | 1.875rem, 400 |
| Título de cards de gráfico        | `text-xl`                       | 1.25rem, 400  |
| Label de logotipo expandido       | `text-2xl font-bold`            | 1.5rem, 700   |
| Label de logotipo colapsado       | `text-[10px] font-bold`         | 10px, 700     |
| Texto de descrição / muted        | `text-sm text-muted-foreground` | 0.875rem      |
| Texto de título de card (shadcn)  | `font-semibold leading-none`    | 600           |
| Texto de card description         | `text-sm text-muted-foreground` | 0.875rem      |
| Badge / Label                     | `text-xs font-medium`           | 0.75rem, 500  |

---

## 4. Border Radius

Definido via variável `--radius: 0.65rem`, com derivados:

| Token         | Valor                         |
| ------------- | ----------------------------- |
| `--radius-sm` | `0.65rem - 4px` → ~`0.4rem`   |
| `--radius-md` | `0.65rem - 2px` → ~`0.525rem` |
| `--radius-lg` | `0.65rem`                     |
| `--radius-xl` | `0.65rem + 4px` → ~`0.9rem`   |

Aplicações relevantes:

- **Cards shadcn:** `rounded-xl` (1rem / `--radius-xl`)
- **Botões:** `rounded-md`
- **Inputs:** `rounded-md`
- **Badges:** `rounded-md`
- **Tabs:** `rounded-lg`
- **Logotipo icônico no SVG:** `rx="16"` (16px)

---

## 5. Espaçamento e Layout

### Estrutura Macro

```
SidebarProvider
├── AppSidebar (floating, collapsible icon)
└── SidebarInset
    ├── [Mobile Header]
    ├── [Connection Banner]
    └── Content Area (bg-gray-50, p-6, flex-col, overflow-hidden)
        └── <Outlet /> → Páginas
```

### Grid e Containers

| Contexto                     | Classe                                                 | Resultado                             |
| ---------------------------- | ------------------------------------------------------ | ------------------------------------- |
| Container de página          | `mx-auto max-w-7xl`                                    | Centralizado, max 80rem               |
| Seção de cards               | `mx-auto max-w-full space-y-8`                         | Full width, espaçamento vertical 2rem |
| Grid de atalhos (Home)       | `grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-3` | Responsivo 1→2→3 colunas              |
| Grid de summary cards        | `grid grid-cols-1 gap-6 md:grid-cols-3`                | Responsivo 1→3 colunas                |
| Espaçamento interno de seção | `space-y-8`                                            | 2rem entre elementos                  |
| Padding de cards             | `p-6`                                                  | 1.5rem                                |
| Gap padrão de formulários    | `gap-6`                                                | 1.5rem                                |

### Sidebar

- **Variante:** `floating` (elevada, separada do conteúdo)
- **Comportamento:** `collapsible="icon"` (colapsa para ícones)
- **Gap entre itens do menu:** `gap-2`
- **Header mobile:** `h-10`, `grid-cols-[1fr_min-content_1fr]`

---

## 6. Componentes UI (shadcn/ui — New York style)

### Button

| Variante      | Aparência                                          |
| ------------- | -------------------------------------------------- |
| `default`     | `bg-primary text-primary-foreground` (verde)       |
| `destructive` | `bg-destructive text-white` (vermelho)             |
| `outline`     | `border bg-background` (borda, fundo transparente) |
| `secondary`   | `bg-secondary text-secondary-foreground`           |
| `ghost`       | Sem fundo, `hover:bg-accent`                       |
| `link`        | `text-primary` com underline no hover              |

| Tamanho   | Altura   | Padding     |
| --------- | -------- | ----------- |
| `default` | `h-9`    | `px-4 py-2` |
| `sm`      | `h-8`    | `px-3`      |
| `lg`      | `h-10`   | `px-6`      |
| `icon`    | `size-9` | —           |

### Badge

| Variante      | Aparência                                |
| ------------- | ---------------------------------------- |
| `default`     | `bg-primary text-primary-foreground`     |
| `secondary`   | `bg-secondary text-secondary-foreground` |
| `destructive` | `bg-destructive text-white`              |
| `outline`     | `text-foreground` com borda visível      |

- Tamanho de texto: `text-xs font-medium`
- Padding: `px-2 py-0.5`
- Ícones internos: `size-3`

### Card

- Estrutura: `CardHeader > CardTitle + CardDescription + CardAction`, `CardContent`, `CardFooter`
- Estilo base: `bg-card rounded-xl border shadow-sm py-6`
- Padding lateral: `px-6` (no header e content)
- Gap entre seções: `gap-6`

### Input

- Altura: `h-9`
- Borda: `border border-input rounded-md`
- Fundo: `bg-transparent` (dark: `bg-input/30`)
- Padding: `px-3 py-1`
- Foco: `ring-ring/50 ring-[3px] border-ring`

### Tabs

- Lista: `bg-muted h-9 rounded-lg p-[3px]`
- Trigger ativo: `bg-background shadow-sm`
- Texto: `text-sm font-medium`

### Header (componente customizado)

```tsx
<Header title="Título da Página">{/* actions opcionais */}</Header>
```

- Layout: `flex flex-col gap-2 md:flex-row md:items-center md:justify-between`
- Botão voltar: `Button variant="ghost" size="icon"` com `FiArrowLeft`
- Título: `text-2xl font-bold`

---

## 7. Iconografia

| Biblioteca                       | Uso                                                                                |
| -------------------------------- | ---------------------------------------------------------------------------------- |
| `lucide-react`                   | Ícones principais da interface (MessageCircle, PanelLeftIcon, Users, etc.)         |
| `react-icons/fi` (Feather Icons) | Ações e navegação (FiArrowLeft, FiLogOut, FiSettings, FiMail, FiSend, FiMic, etc.) |
| `react-icons/fa` (Font Awesome)  | FaWhatsapp no menu lateral                                                         |

Tamanho padrão de ícones em botões: `size-4` (1rem), definido via `[&_svg:not([class*='size-'])]:size-4`.

---

## 8. Navegação e Estrutura de Rotas

```
/auth/login            → Página de login
/auth/select_client    → Seleção de cliente
/auth/logout           → Logout

/:clientId/            → Home (Dashboard / Reports)
/:clientId/whatsapp    → Contatos WhatsApp (Kanban)
/:clientId/email       → Módulo de E-mail
/:clientId/configuration → Configurações
```

### Menu Lateral (AppSidebar)

| Item     | Ícone        | Rota            |
| -------- | ------------ | --------------- |
| E-mails  | `FiMail`     | `email`         |
| WhatsApp | `FaWhatsapp` | `whatsapp`      |
| Clientes | `FiSettings` | `configuration` |
| Sair     | `FiLogOut`   | `/auth/logout`  |

---

## 9. Padrões de Estado e Feedback

| Estado                   | Implementação                                             |
| ------------------------ | --------------------------------------------------------- |
| Loading / Skeleton       | Componente `<Skeleton>` (shadcn) com dimensões explícitas |
| Erro de request          | `toast.error(mensagem)` via `sonner`                      |
| Sucesso de request       | Foco automático no input / reset de estado                |
| Formulário inválido      | `aria-invalid` + `FormMessage` (shadcn)                   |
| Elemento desabilitado    | `disabled:opacity-50 disabled:pointer-events-none`        |
| Gravação de áudio ativa  | `animate-pulse border-destructive text-destructive`       |
| Desconectado (WebSocket) | Banner amarelo absoluto (`bg-yellow-300`) no topo         |

---

## 10. Stack Técnica Relevante

| Camada                   | Tecnologia                                             |
| ------------------------ | ------------------------------------------------------ |
| Framework                | React 19 + TypeScript                                  |
| Build                    | Vite 7                                                 |
| Estilização              | Tailwind CSS v4                                        |
| Componentes base         | shadcn/ui (New York style) + Radix UI                  |
| Variância de componentes | `class-variance-authority` (CVA)                       |
| Tema                     | `next-themes` (suporte a dark mode via classe `.dark`) |
| Ícones                   | Lucide React + React Icons (Feather + Font Awesome)    |
| Formulários              | React Hook Form + Zod                                  |
| Estado servidor          | TanStack Query                                         |
| Gráficos                 | Recharts                                               |
| Roteamento               | React Router v7                                        |
| Notificações             | Sonner                                                 |

---
> Source: [tiago0051/ZapMonitor-Front](https://github.com/tiago0051/ZapMonitor-Front) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
