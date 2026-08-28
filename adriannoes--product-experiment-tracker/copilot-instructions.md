## boas-praticas-frontend

> Padrões para frontend com Next.js (App Router), React, TypeScript, Tailwind e Shadcn/Radix. Alinhado a projetos implantáveis na Vercel; adaptar caminhos se o repo usar ou não a pasta `src/`.


# Boas práticas de frontend (Next.js + Tailwind)

**Escopo:** stack típica da demo e de produtos Next hospedados na **Vercel** (runtime Node/Edge conforme o projeto). Não é exclusivo da Vercel — evite APIs só dela salvo necessidade explícita.

## 1. Stack principal

- **Framework:** Next.js com **App Router**. Evitar roteamento legado em `pages/` para novas features (a menos que o projeto já dependa disso).
- **Linguagem:** TypeScript em modo **strict**, quando o projeto já estiver em TS.
- **Estilo:** **Tailwind CSS** (v3 ou v4, conforme o scaffold) como padrão de estilo; evitar CSS-in-JS pesado salvo decisão explícita do repo.
- **Componentes:** **Shadcn/UI** (primitivos Radix) quando disponíveis no projeto.
- **Ícones:** **Lucide React** (ou a biblioteca de ícones já adotada no repo).

## 2. Server Components (RSC) primeiro

- **Padrão:** componentes são Server Components por omissão.
- **`"use client"`** só quando houver: eventos (`onClick`, `onChange`), hooks (`useState`, `useEffect`) ou APIs do navegador.
- Empurrar lógica de cliente para **baixo** na árvore; manter layout/shell no servidor quando possível.

## 3. Busca de dados e mutações

- **Mutações:** preferir **Server Actions** com `<form action={...}>` (ou padrão equivalente do Next em uso).
- **Leitura em RSC:** `fetch` nativo ou padrão de dados do projeto.
- Evitar **`useEffect` só para carga inicial** de dados; preferir RSC ou, se necessário, biblioteca de dados no cliente (ex. React Query) com justificativa (polling, etc.).

## 4. Estilo (Tailwind)

- **Utilities primeiro;** helper **`cn`** / **`clsx` + `tailwind-merge`** (ou o util já existente no projeto, ex. `cntl`) para classes condicionais.
- Reduzir **valores arbitrários** (`w-[123px]`); preferir **tokens** do tema.
- **Estética “Clean Architect” (referência do curso):** UI minimalista, fundos escuros (preto ou zinc), bordas sutis, alto contraste; sem excesso de neon/gradiente decorativo — salvo pedido explícito de produto.

## 5. Arquitetura de componentes

- Preferir **primitivos de UI do projeto** (Shadcn/Radix) em vez de HTML cru para controles interativos.
- **Caminho dos componentes:** usar a convenção do repo — ex. `@/components/ui`, `components/ui` ou workspace `packages/ui` **se** for monorepo; não assumir só uma estrutura.
- **Composição:** componentes pequenos e com responsabilidade clara.
- **Acessibilidade:** HTML semântico e papéis ARIA onde o design system não cobrir.

## 6. Estrutura de pastas (flexível)

Projetos **com** `src/`:

- `src/app/` — rotas e layouts  
- `src/components/` — UI de feature e composição  
- `src/lib/` — utilitários  
- `src/hooks/` — hooks React  

Projetos **sem** `src/` (como em alguns scaffolds):

- `app/` na raiz para rotas; `components/`, `lib/`, `hooks/` conforme o que o projeto criar.

Ajuste caminhos à árvore real; não forçar `src/` se o app já estiver sem ela.

## 7. Gerenciamento de estado

1. **URL / search params** para estado compartilhável e navegável.  
2. **Servidor:** RSC + cache quando couber.  
3. **Local:** `useState` / `useReducer`.  
4. **Global:** uso mínimo — store leve (ex. Zustand) **só** se o prop drilling ou o fluxo justificarem.

---
> Source: [adriannoes/product-experiment-tracker](https://github.com/adriannoes/product-experiment-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
