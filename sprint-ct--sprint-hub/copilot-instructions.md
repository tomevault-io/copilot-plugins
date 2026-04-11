## sprint-hub

> Este documento fornece diretrizes e contexto para o Gemini atuar no projeto **Sprint Hub**.

# Sprint Hub - Gemini Guidelines

Este documento fornece diretrizes e contexto para o Gemini atuar no projeto **Sprint Hub**.

## 🚀 Tech Stack
- **Framework:** Next.js 16+ (App Router)
- **Linguagem:** TypeScript
- **Estilização:** Tailwind CSS 4 & CSS Variables
- **UI Components:** Shadcn UI (Radix UI)
- **Ícones:** Lucide React & Tabler Icons
- **Banco de Dados:** Prisma ORM
- **Gerenciamento de Formulários:** React Hook Form + Zod
- **Tabelas:** TanStack Table
- **Drag & Drop:** dnd-kit

## 📁 Estrutura de Diretórios
- `src/app`: Rotas e componentes de página.
    - `(private)`: Rotas protegidas (admin, evaluator, startup).
    - `(public)`: Rotas públicas (auth/login).
- `src/components`: Componentes React reutilizáveis.
    - `ui`: Componentes base do Shadcn UI.
- `src/lib`: Utilitários, configurações (ex: `prisma.ts`).
- `prisma`: Schema do banco de dados e migrações.

## 🛠 Padrões de Desenvolvimento
1.  **Componentes:** Utilize `lucide-react` por padrão para ícones. Para ícones específicos não disponíveis, verifique `tabler-icons-react`.
2.  **Server Components:** Prefira Server Components por padrão. Use `"use client"` apenas quando necessário (interatividade, hooks de estado/efeito).
3.  **Formulários:** Devem ser feitos obrigatoriamente com React Hook Form e Zod para validação. Utilize a estrutura de `schema.ts` próxima às páginas de formulário.
4.  **Estilização:** Siga os padrões do Tailwind CSS 4. Evite CSS inline, prefira classes utilitárias.
5.  **Prisma:** Ao realizar alterações no banco, lembre-se de rodar `npx prisma generate`.
6.  **Comunicação:** Ao realizar tarefas notáveis ou mudanças estruturais, explique detalhadamente o plano para o usuário antes de solicitar a implementação.

## 🎨 Design System
- O projeto utiliza um tema dark/light controlado pelo `next-themes`.
- As cores são definidas via variáveis CSS no `globals.css`.

## 📝 Comandos Comuns
- `npm run dev`: Inicia o servidor de desenvolvimento.
- `npm run build`: Cria o build de produção.
- `npx prisma studio`: Abre a interface visual do banco de dados.
- `npx prisma db push`: Sincroniza o schema com o banco de dados (ambiente de dev).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sprint-ct)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sprint-ct)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
