## agendamento

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # servidor de desenvolvimento (Next.js)
npm run build    # build de produção
npm run start    # inicia servidor de produção
npm run hash <senha>  # gera hash bcrypt para cadastrar usuário manualmente
```

Não há test runner configurado. Não há linter configurado (sem eslint/prettier nos scripts).

## Variáveis de Ambiente

Copie `.env.example` para `.env` e preencha:

- `DATABASE_URL` — PostgreSQL principal (propostas, usuários, tabelas legadas `TCE_*`)
- `SCHEDULING_DATABASE_URL` — PostgreSQL de agendamentos (opcional; fallback para `DATABASE_URL`)
- `JWT_SECRET` — string aleatória ≥ 64 chars
- `CRON_SECRET` — segredo para o endpoint `GET /api/cron/cleanup-sessions`
- Variáveis `SMTP_*` — Microsoft 365, STARTTLS na porta 587
- Variáveis `ERP_*`, `NOTIFICATION_RECIPIENTS`, `NEXT_PUBLIC_ENABLE_VINCULO_VI` — configuração por filial (ver abaixo)

## Multi-filial (Viana / Varginha)

O mesmo codebase atende múltiplas filiais, **cada uma como um deploy próprio**
(projeto Vercel próprio) apontando para **seu próprio banco Supabase**. O banco
do portal é single-tenant (sem coluna de filial): senha sequencial, limites do
dia, usuários e regras são isolados por instância naturalmente.

As diferenças entre filiais ficam em `src/shared/infra/config/erpConfig.ts`,
vindas de env (defaults = Viana):

| Env | Viana (default) | Varginha |
|---|---|---|
| `ERP_FILIAL` | `VI` | `VA` |
| `ERP_ID_CLIENTE` / `ERP_ID_OPERACAO` / `ERP_ID_REFERENCIA` | `18` / `2` / `9999` | conforme Protheus |
| `ERP_REQUIRE_PROPOSAL_FORNECEDOR` | `false` | `true` |
| `NOTIFICATION_RECIPIENTS` | equipe Viana | equipe Varginha |
| `NEXT_PUBLIC_ENABLE_VINCULO_VI` | `true` | `false` |

**Origem dos dados por filial** (o ETL entrega as tabelas-espelho no MESMO
formato; a tradução acontece no ETL, não no portal):

- **Viana**: propostas fechadas → `tce_operacoesdiarias`; fornecedores/corretores
  no `clifor`/`corretores`; fornecedor do agendamento resolvido pelo vínculo
  VI × Fornecedor (módulo `vi-fornecedor`).
- **Varginha**: compras do Protheus (SC7) → `tce_operacoesdiarias` **com a coluna
  `id_fornecedor` já resolvida** (C7_FORNECE + C7_LOJA → `clifor.idclifor`);
  SA2 → `clifor` (fornecedores E corretores; `corretores` é uma VIEW sobre
  `clifor.is_corretor`). O módulo `vi-fornecedor` não é usado (menu oculto).
  Schema dos espelhos: `scripts/va-protheus-mirror.sql`. Em Viana, rodar
  `scripts/vi-add-id-fornecedor.sql` (coluna nova, permanece NULL).

**Resolução do fornecedor no agendamento** (POST `/api/scheduling` e
`/api/agendamento-manual`): `body.idFornecedor` > vínculo VI > `proposal.idFornecedor`
(espelho do ERP) > lookup por nome na `clifor`. Com
`ERP_REQUIRE_PROPOSAL_FORNECEDOR=true`, o último fallback é desativado e a
ausência de fornecedor vira erro claro (evita resolver o corretor como
fornecedor silenciosamente).

## Arquitetura

### Clean Architecture + DDD por módulo

Cada módulo em `src/modules/` segue a estrutura:

```
modules/<domínio>/
  domain/
    entities/       # entidades e value objects puros
    errors/         # erros de domínio (estendem DomainError)
    repositories/   # interfaces (IXxxRepository)
    rules/          # regras de negócio puras
  application/
    use-cases/      # classes com responsabilidade única
  infra/
    repositories/   # implementações PostgreSQL (PgXxxRepository)
    mappers/        # conversão DB row → entidade
    index.ts        # instancia e exporta dependências (DI manual)
```

**Módulos existentes:** `user`, `scheduling`, `delivery-rules`, `vi-fornecedor`

### Injeção de dependência

Não há container IoC. Cada módulo tem um `infra/index.ts` (ex.: `userDependencies`, `schedulingDependencies`) que instancia repositórios e use cases manualmente. As rotas de API importam dessas factories.

### Rotas de API

Todas as rotas ficam em `src/app/api/`. O padrão de autenticação em toda rota protegida:

```ts
const user = requireAuth(request)           // ou requireRole / requireAnyRole
if (!isAuthenticated(user)) return user     // user é NextResponse 401/403

// user.userId, user.email, user.roles disponíveis
```

Guards em `src/shared/infra/auth/authGuard.ts`. Token JWT extraído do header `Authorization: Bearer <token>`.

### Autenticação cliente

`src/shared/presentation/auth/` contém:
- `authFetch` — wrapper de `fetch` que injeta o access token e refaz a chamada automaticamente após `/api/auth/refresh` em respostas 401
- `tokenStorage` — mantém o access token em memória (não em localStorage/cookie)
- `userStorage` — persiste info do usuário em `localStorage`

O refresh token fica em cookie HttpOnly; o access token (JWT 15 min) fica só em memória no cliente.

### Banco de dados

Dois clientes `postgres` instanciados em `src/shared/infra/database/`:
- `db` — aponta para `DATABASE_URL`
- `schedulingDb` — aponta para `SCHEDULING_DATABASE_URL` (ou `DATABASE_URL` se ausente)

Tabelas legadas do sistema externo têm prefixo `TCE_` (ex.: `TCE_OPERACOESDIARIAS`, `TCE_PORTAL_FUNCAO`). A tabela central de agendamentos é `tb_agendamentoentrega`. O bloqueio global de agendamentos fica em `tce_portal_bloqueio_agendamento` (linha única). Scripts de schema avulsos ficam em `scripts/*.sql` (idempotentes, rodados manualmente no Supabase).

**Advisory locks** (`pg_advisory_xact_lock`) são usados em `CreateScheduling` e `EditScheduling` para evitar race conditions em agendamentos simultâneos da mesma proposta.

### Validação de agendamento — fluxo completo

0. **Bloqueio global** (kill switch): `ValidateScheduling` checa primeiro a tabela `tce_portal_bloqueio_agendamento` (linha única, `id = 1`). Se `bloqueado = true`, lança `SchedulingGloballyBlockedError` e nenhum agendamento do fluxo normal é criado/editado. Como o agendamento manual **não** passa por `ValidateScheduling`, ele permanece como único caminho ativo (bypass).
1. `ValidateScheduling` (use case, delivery-rules) verifica regras do dia: limite de sacas, tipos permitidos, embalagens, feriado/bloqueio
2. `validateSchedulingCreation` (domain rule, scheduling) verifica data futura e sacas disponíveis na proposta
3. `CreateScheduling` (use case, scheduling) persiste com advisory lock

### Bloqueio global de agendamentos

ADMIN/TRADER podem pausar **todos** os novos agendamentos do portal pela página `/rules` (componente `GlobalBlockPanel`). É distinto do bloqueio **por dia** (`tce_portal_regras_entrega.bloqueado`): é global e independe de data.

- Estado em `tce_portal_bloqueio_agendamento` (`bloqueado`, `mensagem`, `atualizado_em/por`). Módulo `delivery-rules`: `ISchedulingLockRepository` / `PgSchedulingLockRepository` / `ManageSchedulingLock`.
- API `/api/bloqueio-agendamento`: `GET` (qualquer autenticado — brokers leem para exibir o aviso) e `PUT` (`requireAdminOrTrader`).
- Enforcement **no servidor** dentro de `ValidateScheduling` (passo 0). Bloqueia criar (POST) e editar (PUT) no fluxo normal `/api/scheduling`; **cancelar** (DELETE) segue permitido (não passa por `ValidateScheduling`). A UI do broker (banner em `/proposals` + botões desabilitados no `ProposalCard`) é só UX.

### Papéis e controle de acesso

```ts
enum UserRole { CORRETOR = 'CORRETOR', TRADER = 'TRADER', ADMIN = 'ADMIN' }
```

- `CORRETOR` — acessa `/proposals` (cria/edita/cancela agendamentos)
- `TRADER` — acessa `/dashboard`, `/rules`
- `ADMIN` — acesso total + `/brokers`, `/vinculo-vi-fornecedor`

O `AppLayout` (`src/app/(app)/layout.tsx`) redireciona o cliente para a rota correta conforme o papel. As rotas de API validam papel no servidor via `requireRole` / `requireAnyRole`.

**Agendamento manual** (`/agendamento-manual`, APIs `/api/agendamento-manual` e `/api/agendamento-manual/search`): exclusivo de ADMIN/TRADER (`requireAdminOrTrader`). Permite buscar qualquer compra por referência (ex.: `VI260554`) ou código e agendar, editar ou cancelar a entrega de **qualquer** agendamento (independente de quem o criou) **sem** passar pelo `ValidateScheduling` — bypass intencional dos limites do dia (sacas, embalagem, tipo, feriado/bloqueio), exclusivo dessa rotina. A validação de saldo de sacas da compra permanece, e as sacas agendadas contam normalmente no saldo do dia para o fluxo dos brokers. **Datas no passado** são permitidas nesta rotina (e só nela), tanto na **criação** quanto na **edição** (`allowPastDelivery` em `CreateScheduling` e `EditScheduling`) — para registrar/ajustar retroativamente uma entrega já ocorrida e consumir o saldo da compra. Vale qualquer data passada (não só manter a original): ex.: agendado 1500, entregue 500, edita para 500 e libera 1000 de saldo. O fluxo normal do broker sempre exige data futura. Os e-mails dessa rotina vão para o broker dono da compra (`findEmailsByAlias`), não para o usuário logado — **exceto** quando a data de entrega já passou (`isDeliveryDateInPast`): agendamentos/edições retroativos **não** disparam e-mail.

No fluxo normal (`/api/scheduling` PUT/DELETE), a posse do agendamento é verificada por: criado pelo próprio usuário **ou** compra pertencente a um alias do broker autenticado — assim o corretor consegue editar/cancelar agendamentos criados por ADMIN/TRADER via agendamento manual.

### Grupos de café

- `'A'` = Arábica
- `'C'` = Conilon

Embalagens: `'BIG_BAG'`, `'GRANEL'`, `'SACARIA'` (no banco) / `'BIG BAG'`, `'GRANEL'`, `'SACARIA'` (na API externa — atenção ao espaço).

### Fuso horário

Toda validação de data usa `America/Sao_Paulo` via `getTodayInSaoPaulo()` em `src/modules/scheduling/domain/rules/schedulingRules.ts`. Datas do banco são salvas e comparadas em UTC; converter ao exibir.

### Email

`NodemailerEmailService` em `src/shared/infra/email/`. Disparado pelos use cases de scheduling (criação, edição, cancelamento). Usa templates string em PT-BR.

### Status de agendamento

`PENDENTE` → `CONCLUIDO` | `CANCELADO`  
Edição não altera o registro: marca o original como `HISTORICO_EDICAO` e cria um novo `PENDENTE`. A **senha é preservada** na nova versão (não é regerada) — `EditScheduling` repassa `original.senha` ao `editAtomic`; só a criação gera senha nova (MAX+1).

### Segurança

Security headers definidos em `next.config.ts` (CSP, HSTS, X-Frame-Options, etc.). Rate limiting no endpoint de login em `src/shared/infra/auth/rateLimiter.ts`. Em desenvolvimento, a CSP inclui `'unsafe-eval'` para o HMR do Next.js.

---
> Source: [realCafeMiguel/agendamento](https://github.com/realCafeMiguel/agendamento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
