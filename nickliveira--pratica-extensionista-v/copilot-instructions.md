## pratica-extensionista-v

> - Next.js 14 com App Router

# Regras do Projeto - Sistema de Cupons de Desconto

## Stack Tecnológico
- Next.js 14 com App Router
- TypeScript
- Prisma ORM
- PostgreSQL
- NextAuth.js para autenticação
- Tailwind CSS para estilização

## Convenções de Código

### TypeScript
- Usar TypeScript em todos os arquivos
- Definir tipos explícitos quando necessário
- Evitar `any`, preferir tipos específicos

### Componentes React
- Usar 'use client' apenas quando necessário (interatividade)
- Preferir Server Components quando possível
- Nomear componentes com PascalCase

### Estilização
- Usar Tailwind CSS exclusivamente
- Seguir padrão de cores do projeto (azul, verde, cinza)
- Manter responsividade (mobile-first)

### API Routes
- Sempre validar entrada com Zod
- Sempre verificar autenticação quando necessário
- Retornar status HTTP apropriados
- Tratar erros adequadamente

### Banco de Dados
- Usar Prisma para todas as operações
- Sempre usar transações para operações múltiplas
- Seguir o schema existente em prisma/schema.prisma

## Estrutura de Arquivos

```
app/
  api/              # API Routes
  (paginas)/        # Páginas do app
components/         # Componentes reutilizáveis
lib/               # Utilitários e configs
prisma/            # Schema e migrations
types/             # Definições TypeScript
```

## Padrões de Autenticação

- Usar NextAuth.js
- Verificar sessão com getServerSession
- Proteger rotas com middleware.ts
- Diferenciar ASSOCIADO e COMERCIANTE

## Validações

- Validar CPF com algoritmo completo
- Validar CNPJ com algoritmo completo
- Validar datas (início <= fim)
- Validar percentuais (1-100)
- Sempre validar no servidor

## Segurança

- Nunca expor senhas
- Sempre usar bcrypt para hash
- Validar permissões (comerciante vs associado)
- Sanitizar inputs
- Usar prepared statements (Prisma já faz isso)

## Mensagens de Erro

- Português do Brasil
- Claras e específicas
- Orientar o usuário sobre o que fazer

## Git Commits

Usar prefixos:
- feat: Nova funcionalidade
- fix: Correção de bug
- docs: Documentação
- style: Formatação
- refactor: Refatoração
- test: Testes

Exemplo: "feat: adicionar filtro por categoria"

## Performance

- Usar Server Components quando possível
- Otimizar queries do Prisma (include apenas necessário)
- Implementar loading states
- Evitar re-renders desnecessários

## Acessibilidade

- Labels em todos inputs
- Mensagens de erro associadas a campos
- Contraste adequado de cores
- Navegação por teclado funcional

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nickliveira) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
