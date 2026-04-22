## emma-dev

> EMMA e uma plataforma de gestao para clinica de saude/bem-estar e escola de massagem terapeutica.

# EMMA — Instrucoes para Agentes

## Visao Geral
EMMA e uma plataforma de gestao para clinica de saude/bem-estar e escola de massagem terapeutica.
O projeto tem **frontend** (React) e **backend** (Laravel API) — estamos a desenvolver **apenas o frontend**.

## Stack Frontend (NAO alterar)
- React 19 + Vite 5
- Tailwind CSS 3.4 (dark mode via classe `dark`)
- Axios (cliente HTTP)
- React Hook Form + @hookform/resolvers + Zod (formularios e validacao)
- React Router DOM (routing SPA)
- Lucide React (icones SVG)

## Regra Fundamental
**ZERO logica de negocio no frontend.** O frontend apenas:
- Apresenta dados recebidos da API
- Valida inputs do utilizador (UX validation com Zod)
- Envia dados para a API
- Exibe respostas/erros da API

Calculos, regras de negocio, autorizacao — tudo e feito pelo backend.

## Convencoes de Codigo
- **Componentes:** PascalCase, um por ficheiro, funcionais (sem class components)
- **Hooks:** prefixo `use` (useAuth, useStudents, etc.)
- **Services:** camelCase, um ficheiro por recurso API
- **Schemas:** camelCase, Zod schemas para validacao de formularios
- **Imports:** usar alias `@/` para `src/` (ex: `import { Button } from '@/components/ui/Button'`)

## Design System
- **Light Mode:** emerald-800 primary, slate-50 bg, white cards, rounded-xl
- **Dark Mode:** emerald-400/600 primary, slate-950 bg, slate-900 cards
- **Fonte:** Inter (Google Fonts)
- **Icones:** Lucide React (nao usar outros icon packs)
- **Dark mode obrigatorio** em TODOS os componentes (classes `dark:`)

## Estrutura de Pastas
```
frontend/src/
  components/ui/       — Design system (Button, Input, Card, Modal, etc.)
  components/layout/   — Sidebar, Header, PageContainer
  components/feedback/ — Toast, Loading, ErrorMessage, AlertBanner
  pages/               — Uma pasta por modulo (auth/, dashboard/, students/, etc.)
  hooks/               — Custom hooks
  services/api/        — Camada de comunicacao com API
  schemas/             — Schemas Zod
  contexts/            — React Contexts (Auth, Theme)
  utils/               — Funcoes utilitarias puras
  constants/           — Rotas, enums, config
```

## API
- **Dev:** `http://localhost:8000/api`
- **Producao:** `https://beautiful-teal-horse.185-118-115-20.cpanel.site/api`
- **Auth:** Laravel Sanctum (Bearer token)
- Documentacao completa em `.obsidian/03-API/endpoints.md.md`

## Documentacao
- Toda a documentacao do projeto esta em `.obsidian/`
- Sempre documentar decisoes arquiteturais em `.obsidian/`
- Consultar `.obsidian/06-Prompts/` para tarefas passo a passo

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/erictarouquela) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
