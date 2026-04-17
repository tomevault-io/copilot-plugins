## investia

> Você está trabalhando no InvestIA, um SaaS de análise de investimentos.

# InvestIA - Cursor Rules

# Projeto
Você está trabalhando no InvestIA, um SaaS de análise de investimentos.
Stack: NestJS + Prisma + PostgreSQL + Redis + Docker

# Arquitetura
- Use Clean Architecture com camadas: Controllers → Use Cases → Entities ← Repositories
- Cada módulo deve ter: application/use-cases, domain/entities, infrastructure/repositories
- Dependências sempre apontam para dentro (Dependency Rule)

# Convenções de Código

## TypeScript/NestJS
- Use Cases: VerbNounUseCase (ex: CreateAssetUseCase)
- Services: NounService (ex: MarketDataService)  
- DTOs: VerbNounInput/NounOutput (ex: CreateAssetInput)
- Sempre tipar retornos de funções
- Evite `any` - use `unknown` se necessário

## Prisma
- Tabelas em snake_case plural (@@map)
- Colunas em snake_case (@map)
- Sempre use UUIDs para IDs
- Campos padrão: id, createdAt, updatedAt

## Testes
- Estrutura AAA: Arrange-Act-Assert
- Nome: should [behavior] when [condition]
- Mock repositórios, não use banco real em unit tests

# Docker
- Ambiente 100% Dockerizado - nunca instale nada localmente
- Comandos via: docker compose exec app [comando]
- API roda na porta 3001

# Documentação
- Consulte .ai/docs/ para decisões arquiteturais
- Consulte .ai/agents/ para guidelines por área
- Atualize .ai/plan/ ao completar tarefas

# NÃO FAÇA
- Lógica de negócio em Controllers
- SQL raw sem necessidade extrema
- Imports de infraestrutura em Entities
- Secrets hardcoded
- Mudanças de schema sem migration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/valter-tonon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
