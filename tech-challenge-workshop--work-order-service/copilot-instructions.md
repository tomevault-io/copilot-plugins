## work-order-service

> Microserviço de **Atendimento e Ordens de Serviço** do Tech Challenge FIAP (Fase 4) — sistema de gestão de oficina mecânica em arquitetura de microserviços. Os requisitos completos do desafio estão em `../REQUISITOS.md` (fora deste repo, no diretório da organização).

# work-order-service

Microserviço de **Atendimento e Ordens de Serviço** do Tech Challenge FIAP (Fase 4) — sistema de gestão de oficina mecânica em arquitetura de microserviços. Os requisitos completos do desafio estão em `../REQUISITOS.md` (fora deste repo, no diretório da organização).

## Responsabilidades deste serviço

É o dono do ciclo de vida da OS e dos cadastros mestres:

- CRUD de **clientes** (CPF/CNPJ validados) e **veículos** (placa, marca, modelo, ano)
- CRUD do **catálogo de serviços** da oficina (com preços)
- **Abertura de OS** (retorna ID único, grava snapshot de itens/preços)
- **Máquina de status** da OS + histórico de transições
- Listagem ordenada de OS e consulta de status pelo cliente
- **Orquestrador da Saga** do fluxo transacional da OS
- Notificação ao cliente a cada mudança de status
- Métrica de tempo médio de execução (a partir do histórico)

**Fora do escopo (outros microserviços):** orçamento e pagamento (`billing`, integra Mercado Pago), execução/diagnóstico e **estoque de peças** (`execution`). Nunca acessar o banco de outro serviço — integração só via eventos RabbitMQ ou REST síncrono.

## Stack

- **NestJS 11** + TypeScript, **pnpm** (não usar npm/yarn)
- **Processo único**: HTTP no `main.ts` + mensageria via `RabbitMqBus` (`shared/messaging`), que conecta no boot (`OnApplicationBootstrap`) a um **topic exchange `saga`** e faz bind da fila do serviço às routing keys assinadas. Não usar o transporte RMQ nativo do Nest (roteia pela pattern interna, não pela routing key). Não criar entrypoint separado de worker por enquanto.
- **PostgreSQL** via **Prisma** (schema em `prisma/schema.prisma`)
- **RabbitMQ** via `@nestjs/microservices` + `amqplib` (eventos da Saga)
- **Jest** (unit + e2e), cobertura mínima **80%** (exigência do desafio)
- Swagger (`@nestjs/swagger`) — toda rota nova deve aparecer documentada

## Comandos

```bash
pnpm install              # dependências
pnpm start:dev            # dev com watch
pnpm build                # build de produção
pnpm test                 # testes unitários
pnpm test:cov             # cobertura (manter ≥80%)
pnpm test:e2e             # testes e2e (requer docker compose up antes)
pnpm lint                 # eslint --fix
pnpm format               # prettier
docker compose up -d      # Postgres + RabbitMQ locais
npx prisma migrate dev    # criar/aplicar migration
```

Antes de considerar qualquer tarefa concluída: `pnpm lint:check && pnpm test && pnpm build`.

## Arquitetura (Clean Architecture, módulos por contexto)

```
src/
├── modules/
│   ├── customers/
│   │   ├── domain/          # entidades, value objects, regras de negócio puras
│   │   ├── application/     # use cases + ports (interfaces de repositório/gateway)
│   │   ├── presentation/    # controllers HTTP, DTOs (request/response), consumers de eventos
│   │   └── infra/           # repositórios Prisma, adapters, publishers RabbitMQ
│   ├── vehicles/
│   ├── catalog/
│   ├── work-orders/
│   └── saga/                # orquestrador, handlers de eventos, compensações
└── shared/                  # config, messaging (RabbitMQ), guards, base classes
```

**Regras de dependência (invioláveis):**

- `domain/` não importa nada de Nest, Prisma ou RabbitMQ — TypeScript puro.
- `application/` depende só de `domain/` e define ports (interfaces); não conhece Prisma/HTTP.
- `presentation/` é a porta de entrada: controllers HTTP e consumers de eventos traduzem DTOs em chamadas de use cases. Não contém regra de negócio nem acessa Prisma/repositórios diretamente.
- `infra/` é a porta de saída: implementa os ports (repositórios Prisma, publishers RabbitMQ, clients HTTP para outros serviços). Único lugar com Prisma Client.
- Use cases recebem dependências por injeção via tokens dos ports, nunca instanciam adapters.
- DTOs de `presentation/` nunca vazam para `application/` — use cases recebem/retornam tipos próprios (input/output models).

## Linguagem ubíqua (código em inglês)

| Domínio (PT) | Código (EN) |
|---|---|
| Ordem de Serviço (OS) | `WorkOrder` |
| Cliente | `Customer` |
| Veículo | `Vehicle` |
| Serviço (catálogo) | `RepairService` (evita colisão com *service* do Nest) |
| Peça/Insumo | `Part` / `Supply` |
| Orçamento | `Quote` |

## Regras de negócio críticas

**Status da OS** (enum `WorkOrderStatus`):
`RECEIVED → IN_DIAGNOSIS → AWAITING_APPROVAL → IN_EXECUTION → FINISHED → DELIVERED` (+ `CANCELLED` via compensação da Saga). Transições só acontecem por eventos/use cases — nunca update direto de status. Toda transição gera registro no histórico (base do tempo médio de execução).

**Listagem de OS:**
- Ordenar por status: `IN_EXECUTION > AWAITING_APPROVAL > IN_DIAGNOSIS > RECEIVED`; dentro do status, mais antigas primeiro.
- `FINISHED` e `DELIVERED` não aparecem na listagem (**exclusão lógica**, nunca DELETE físico).

**Abertura de OS:**
- Preços de peças são consultados no `execution` service via REST síncrono no momento da criação; serviços vêm do catálogo próprio. Gravar **snapshot** de itens+preços na OS (billing calcula o orçamento a partir do snapshot no evento).

**Validações:** CPF/CNPJ e placa validados no domain (value objects), não apenas no DTO.

## Saga (orquestrada, este serviço é o orquestrador)

Fluxo feliz: `WorkOrderCreated → PartsReserved → QuoteGenerated → QuoteApproved → PaymentConfirmed → ExecutionStarted → ExecutionCompleted → Delivered`.

Falha em qualquer passo dispara compensações na ordem inversa (liberar reserva de peças, cancelar quote, estornar pagamento, `CANCELLED` na OS). Todo evento novo deve ter: nome em inglês no passado (`QuoteApproved`), payload versionado documentado, handler idempotente (eventos podem ser reentregues).

## Autenticação

JWT emitido pela Lambda (repo `tech-lambda`) — este serviço **valida** o token, não emite. Roles: `admin` (CRUDs e listagens administrativas) e `customer` (consulta de status/histórico das próprias OS). Rotas administrativas sempre com guard de role.

## Testes

- **Todos os testes vivem em `tests/` na raiz**, espelhando a estrutura de `src/` (ex.: `tests/modules/customers/domain/document.spec.ts`). E2E em `tests/e2e/`. Nunca colocar `.spec.ts` dentro de `src/`.
- TDD: escrever o teste do use case antes da implementação.
- Domain e application testados sem Nest/DB (mocks dos ports).
- Pelo menos um fluxo completo com **BDD** (exigência do desafio).
- Cobertura ≥80% é gate de CI — não fazer merge abaixo disso.

---
> Source: [tech-challenge-workshop/work-order-service](https://github.com/tech-challenge-workshop/work-order-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
