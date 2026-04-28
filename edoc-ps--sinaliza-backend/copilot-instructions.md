## sinaliza-backend

> **Projeto:** Repositório de Cadastro de Sinais para Pessoas Surdas

# CLAUDE.md — repo-backend

## Contexto do Projeto

**Projeto:** Repositório de Cadastro de Sinais para Pessoas Surdas  
**Instituição:** IFMG  
**Tipo:** Trabalho de Conclusão de Curso (TCC)  
**Sprint Atual:** Modulação (Prototipagem)

### Objetivo
Criar uma plataforma web que funcione como repositório de cadastro de sinais (Libras/linguagem de sinais), permitindo que pessoas surdas registrem, compartilhem e consultem sinais de forma centralizada.

### Escopo Atual (Sprint de Modulação)
- Fase de prototipagem com entidades mínimas
- Foco em autenticação básica via login de usuário
- **Entidade principal:** Apenas `User` para autenticação e identificação
- Validações e estrutura essencial para futuros módulos

---

## Visão geral

Backend NestJS em estágio inicial de desenvolvimento. Atualmente contém a estrutura base gerada pelo CLI do NestJS e o schema Prisma com o modelo `User` para autenticação.

## Stack

- **Framework:** NestJS 11 (TypeScript)
- **ORM:** Prisma 7 com PostgreSQL
- **Runtime:** Node.js
- **Testes:** Jest + Supertest

## Banco de dados

- **Provider:** PostgreSQL
- **Container Docker:** `repo-postgres` (porta `5432`)
- **Credenciais padrão (dev):** `postgres / postgres`, database `repodb`

### Subir o banco

```bash
docker-compose up -d
```

### Migrations e client

```bash
npx prisma migrate dev   # cria e aplica migration
npx prisma generate      # gera o Prisma Client
npx prisma studio        # abre o Prisma Studio
```

## Comandos principais

```bash
npm run start:dev   # servidor em modo watch (desenvolvimento)
npm run build       # compila para dist/
npm run start:prod  # executa build de produção
npm run lint        # eslint com autofix
npm run test        # testes unitários
npm run test:e2e    # testes end-to-end
npm run test:cov    # cobertura de testes
```

## Schema atual (Prisma)

Modelo `User`:

| Campo       | Tipo       | Observação                  |
|-------------|------------|-----------------------------|
| id          | Int        | PK, autoincrement           |
| name        | String     |                             |
| email       | String?    | unique                      |
| birthdate   | DateTime?  |                             |
| password    | String     |                             |
| phone       | String?    |                             |
| avatar      | String?    |                             |
| status      | Boolean    | default: true               |
| role        | Role       | enum: USER \| ADMIN         |
| createdAt   | DateTime   | default: now()              |
| updatedAt   | DateTime   | @updatedAt                  |

## Estrutura de pastas

```
src/
  app.module.ts       # módulo raiz
  app.controller.ts   # controller raiz (placeholder)
  app.service.ts      # service raiz (placeholder)
  main.ts             # bootstrap da aplicação
prisma/
  schema.prisma       # schema do banco de dados
```

## Variáveis de ambiente

Crie um arquivo `.env` na raiz com pelo menos:

```env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/repodb"
PORT=3000
```

## Convenções

- Módulos ficam em `src/modules/<nome>/` (padrão NestJS: controller, service, module, dto)
- DTOs usam `class-validator` para validação
- Porta padrão: `3000`

## Roadmap (Futuro)

### Próximos módulos esperados
- **Signals/Sinais:** Cadastro, busca e gerenciamento de sinais
- **Categories/Categorias:** Organização de sinais por categoria
- **Videos:** Upload e armazenamento de vídeos de sinais
- **Comments/Comentários:** Sistema de comentários nos sinais
- **Permissions/Permissões:** Controle de acesso granular

### Considerações para TCC
- Manter a arquitetura escalável para adicionar novos módulos
- Documentar APIs com Swagger/OpenAPI
- Implementar testes unitários e e2e conforme novos módulos surgirem
- Considerações de acessibilidade no backend (headers, convenções, etc)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EdOc-PS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
