## senhas

> Last Updated: 2026-06-27 (sessão 2)

# AGENTS.md - Guia Operacional para Agentes de IA

Last Updated: 2026-06-27 (sessão 2)
Project: Senhas / GiraHub - Multi-Tenant SaaS para emissão de tickets
Repository: leonfpontes/Senhas
Default Branch: master
Working Branch (atual): master
VPS: 76.13.231.19 (Hostinger) — projeto clonado em /opt/senhas

Este arquivo define como agentes de IA devem entender o sistema e como agir ao implementar mudanças com seguranca, qualidade e consistencia arquitetural.

---

## 1) Objetivo do Produto

Senhas e um SaaS multi-tenant para emissao e gestao de tickets (senhas) para atendimento em giras.

Principais modulos:
- API publica de emissao e reenvio de senha.
- Painel admin do tenant (giras, porta, tickets, analytics, config, auditoria).
- Painel platform (super admin) para gestao de tenants, usuarios globais, billing e feature flags.

---

## 2) Mapa Rapido do Monorepo

- backend/: FastAPI + SQLAlchemy async + Alembic + testes Pytest.
- frontend/: Next.js + TypeScript + Material UI + Jest/RTL.
- packages/shared-types: contratos tipados compartilhados.
- packages/shared-ui: componentes e tema compartilhado.
- docs/: arquitetura, API, auth, multi-tenancy, deploy e testes.
- e2e/: cenarios E2E.
- load_tests/: testes de carga.
- security/: scripts/checklist de seguranca.

---

## 3) Arquitetura e Regras Nao Negociaveis

### 3.1 Multi-tenancy (obrigatorio)

Toda operacao sensivel deve respeitar isolamento por tenant em 3 camadas:
1. JWT carrega tenant_id no payload.
2. Middleware coloca tenant_id em request.state.
3. Repository filtra por tenant_id em query.

Regra critica:
- Nenhuma leitura/escrita de entidade de tenant sem filtro explicito de tenant_id.
- Evite bypass de repository para logica de negocio, exceto quando realmente necessario e com filtro de tenant preservado.

### 3.2 Auth e autorizacao

- Roles principais: SUPER_ADMIN, ADMIN, OPERATOR.
- Endpoints admin so para escopo do tenant atual.
- Endpoints platform so para super admin (escopo global).

**Fluxo de autenticacao via cookie HttpOnly (desde 2026-06-27):**
- Login seta 3 cookies: `access_token` (HttpOnly, Secure, SameSite=Strict), `refresh_token` (HttpOnly), `auth_state=1` (nao-HttpOnly — legivel por JS para verificar login).
- `/auth/refresh` implementado: le `refresh_token` do cookie, valida com `decode_refresh_token` (requer `type=refresh`), emite novo access + rotaciona refresh.
- `jwt_middleware` extrai token do header `Authorization: Bearer` primeiro (impersonacao via sessionStorage), depois fallback para cookie `access_token`.
- `jwt_middleware` public_paths inclui `/auth/refresh`, `/auth/forgot-password`, `/auth/reset-password`.
- Frontend usa `withCredentials: true` no axios — nao ha token no header para sessoes normais.
- Impersonacao usa sessionStorage e header Bearer — fluxo preservado separado.
- `hasAuthToken()` checa: `sessionStorage.getItem('access_token')` OR `document.cookie.includes('auth_state=1')` OR `localStorage.getItem('user')`.
- Logout DEVE chamar `POST /api/v1/auth/logout` para limpar cookies no servidor.

### 3.3 Grupos de Permissao — OBRIGATORIO em toda funcionalidade

O sistema implementa RBAC fino via `PermissionGroup` / `GroupPermission`. Todo endpoint admin e toda
tela admin DEVEM respeitar esse sistema. Ignorar esse requisito e considerado um bug critico de seguranca.

#### Backend — todo novo endpoint admin precisa de:

```python
from src.models import PermissionFeature
from src.api.dependencies import require_group_permission

@router.get("/recurso", dependencies=[Depends(require_group_permission(PermissionFeature.FEATURE, "view"))])
@router.post("/recurso", dependencies=[Depends(require_group_permission(PermissionFeature.FEATURE, "insert"))])
@router.put("/recurso/{id}", dependencies=[Depends(require_group_permission(PermissionFeature.FEATURE, "edit"))])
@router.delete("/recurso/{id}", dependencies=[Depends(require_group_permission(PermissionFeature.FEATURE, "delete"))])
```

Acoes mapeadas por tipo de endpoint:
- GET (listagem/detalhe) → "view"
- POST (criar/registrar) → "insert"
- PUT/PATCH (atualizar) → "edit"
- DELETE (remover) → "delete"

Rotas existentes e suas features:
- Giras, Porta (door_control) → `PermissionFeature.GIRAS` / `PermissionFeature.PORTA`
- Tickets, tickets_bulk, validate_bulk, email_resend → `PermissionFeature.TICKETS`
- Mediuns → `PermissionFeature.MEDIUNS`
- Associados → `PermissionFeature.ASSOCIADOS`
- Usuarios → `PermissionFeature.USUARIOS`
- Estoque → `PermissionFeature.ESTOQUE`
- Mensalidades (financeiro/config/resumo/relatorio) → `PermissionFeature.FINANCEIRO`
- Contas a Pagar/Receber, Fluxo de Caixa, Config Financeira → `PermissionFeature.CONTAS_FINANCEIRAS`
- Configuracoes do Tenant → `PermissionFeature.CONFIGURACOES`
- Auditoria → `PermissionFeature.AUDITORIA`
- Analytics → `PermissionFeature.ANALYTICS`
- Relatorio de Gira / exports CSV → `PermissionFeature.RELATORIO_GIRA`
- Cursos Presenciais / Sites → `PermissionFeature.CURSOS_PRESENCIAIS`

Para nova feature sem equivalente existente:
1. Adicionar valor ao enum `PermissionFeature` em `backend/src/models/permission_groups.py`.
2. Criar migracao Alembic para adicionar o valor ao tipo ENUM no banco (`ALTER TYPE ... ADD VALUE`).
3. Adicionar entrada em `frontend/src/constants/permissionFeatures.ts` (type union + FEATURE_LABELS com label e group).
4. Mapear no `permission_service.py` se a feature requer restricao de plano.

#### Frontend — toda nova tela admin precisa de:

```tsx
import { usePermissions } from '@/hooks/usePermissions';
import { useSubscription } from '@/hooks/useSubscription';

const { can: canGroup } = usePermissions();    // grupo de permissao (RBAC)
const { can } = useSubscription();             // feature flag de plano

// Gate de plano (se a feature tiver restricao de plano)
if (!can('nome_da_feature_no_plano')) {
  return <UpgradePrompt ... />;
}

// Gate de grupo de permissao (OBRIGATORIO)
if (!canGroup('feature_enum_value', 'view')) {
  return <Alert severity="warning">Sem permissao para visualizar.</Alert>;
}

// Guard de acoes destrutivas/escrita
const canInsert = canGroup('feature_enum_value', 'insert');
const canEdit   = canGroup('feature_enum_value', 'edit');
const canDelete = canGroup('feature_enum_value', 'delete');
```

Regras de UI:
- Botoes de criar/editar/excluir devem ser ocultados (nao apenas desabilitados) quando sem permissao.
- Fetchers que chamam endpoints protegidos devem checar `canGroup` antes do request.
- A coluna de acoes em tabelas deve ser omitida quando `canInsert && canEdit && canDelete` sao todos false.

#### Checklist especifico para grupos de permissao

Ao criar ou modificar qualquer funcionalidade:
- [ ] Backend: todos os endpoints novos tem `require_group_permission` com feature e acao corretos.
- [ ] Backend: feature existente ou nova foi criada no enum `PermissionFeature`.
- [ ] Frontend: hook `usePermissions` importado e `canGroup` checado antes de fetch e render de acoes.
- [ ] Frontend: tela exibe mensagem de "sem permissao" (nao erro 403) quando grupo nao autoriza view.
- [ ] Frontend: `permissionFeatures.ts` atualizado se nova feature foi criada (label + group).
- [ ] Rotas de sistema (health, billing, subscription_info, permission_groups) sao excecao — nao precisam de guard de grupo.

### 3.3 Integridade de emissao de senha

- Emissao deve permanecer atomica/confiavel sob concorrencia.
- Em contadores de senha, use padroes com lock transacional (ex.: SELECT FOR UPDATE) ja adotados no projeto.

---

## 4) Convencoes de Implementacao

### 4.1 Backend

- Stack alvo: Python 3.11+, FastAPI, SQLAlchemy 2 async, Pydantic v2.
- Fluxo padrao:
	- Modelo ORM em backend/src/models.
	- Regra de acesso em backend/src/repositories.
	- Endpoint em backend/src/api/v1/{public|admin|platform|auth}.
	- Migracao Alembic em backend/alembic/versions.
	- Testes em backend/tests.
- Nao quebrar contratos de resposta sem atualizar frontend, shared-types e docs.
- Erros HTTP devem ser claros, consistentes e com status code adequado.

### 4.2 Frontend

- Stack alvo: Next.js + TypeScript + MUI.
- Preferir componentes reutilizaveis e hooks existentes.
- Evitar duplicacao de chamadas API; centralizar em services/client.
- Garantir estado de loading, erro e sucesso em telas administrativas.
- Responsividade obrigatoria (desktop e mobile).

### 4.3 Banco e migracoes

- Toda mudanca de schema exige migracao Alembic.
- Migracoes devem ser reversiveis (downgrade coerente sempre que possivel).
- Nomes de colunas/indices/constraints devem ser claros e estaveis.
- **OBRIGATORIO antes de criar qualquer migracao**: verificar se ha multiplas heads com `alembic heads`. Se houver mais de uma, criar merge revision primeiro (`alembic merge heads -m "merge"`) antes de adicionar nova migracao. Nunca criar duas migracoes com o mesmo `down_revision` em branches diferentes sem merge.

### 4.4 Convencao de Enums SQLAlchemy 2.0 (CRITICA)

O SQLAlchemy 2.0 usa o `.name` do enum Python para lookup no banco por padrao. Para enums com valores
lowercase no banco, e obrigatorio usar `values_callable=lambda x: [e.value for e in x]` no SQLEnum.

**Enums com valores lowercase no banco** (obrigatorio `values_callable`):
- `user_role`: super_admin, admin, operator
- `ticket_status`: emitted, called, completed, cancelled, no_show
- `subscription_status`: active, suspended, cancelled, expired
- `invoice_status`: draft, sent, paid, overdue, cancelled
- `audit_action`: create, read, update, delete, login, logout, token_refresh, TENANT_DELETED
- `estoque_movimentacao_tipo`: entrada, saida

**Enums com valores UPPERCASE no banco** (NAO usar values_callable):
- `plan_type`: FREE, BASIC, PRO, PREMIUM
- `mensalidade_status`: PENDENTE, PAGO, ISENTO
- `site_status`, `site_section_type`: UPPERCASE

**Regra para novos enums**: decidir antes de criar se serao lowercase ou UPPERCASE e manter consistencia.
Misturar (DB uppercase + values_callable, ou DB lowercase sem values_callable) causa LookupError em runtime.

---

## 5) Politica de Seguranca (OBRIGATORIA)

### 5.1 Proibido commitar segredos

Nunca subir no repositorio:
- Senhas reais.
- JWTs reais.
- API keys reais (Brevo, Resend, etc.).
- Connection strings reais com credenciais.
- Arquivos .env com valores reais.

Permitido:
- Placeholders explicitos (ex.: your_api_key_here).
- Dados de teste claramente nao produtivos.

### 5.2 Redacao segura em codigo/docs

- Ao documentar, use exemplos anonimizados/placeholders.
- Nunca logar credenciais, tokens ou payloads sensiveis completos.
- Se detectar segredo no historico da branch em trabalho, interrompa fluxo de push/PR e sanitize antes.

---

## 6) Qualidade, Testes e Validacao

Antes de concluir implementacao, executar validacoes proporcionais ao impacto:

Backend:
- Testes unitarios/integracao afetados.
- Verificacao de imports, tipagem e lint (quando configurado).

Frontend:
- Testes de componentes/paginas afetadas.
- Build/typecheck quando mudancas forem amplas.

Fluxo minimo recomendado por mudanca:
1. Implementar.
2. Rodar testes alvo.
3. Revisar diff para regressao e segredos.
4. Atualizar docs quando contrato/comportamento mudar.

---

## 7) Boas Praticas de Desenvolvimento para Agentes

- Fazer mudancas pequenas e focadas por commit sempre que possivel.
- Preservar padroes existentes do repositorio.
- Evitar refactors amplos sem necessidade funcional clara.
- Manter compatibilidade retroativa quando viavel.
- Explicar no PR o que mudou, risco e como validar.
- Se encontrar alteracoes inesperadas nao relacionadas durante a tarefa, pausar e alinhar com o usuario.

---

## 8) Checklist de Implementacao (Use Sempre)

Antes de abrir PR, confirme:
- [ ] Isolamento multi-tenant preservado.
- [ ] Nao ha segredo hardcoded nos arquivos alterados.
- [ ] Migracao criada/aplicavel para mudanca de schema.
- [ ] `alembic heads` retorna exatamente UMA head (sem divergencias).
- [ ] Testes relevantes executados e passando.
- [ ] Docs atualizadas (API, comportamento ou operacao).
- [ ] Frontend funciona em desktop/mobile para a funcionalidade alterada.
- [ ] Logs/erros sem vazamento de dados sensiveis.
- [ ] **Grupos de permissao**: todos os novos endpoints tem `require_group_permission` com feature e acao corretos (ver secao 3.3).
- [ ] **Grupos de permissao**: frontend usa `canGroup` para bloquear a tela (view) e ocultar acoes (insert/edit/delete).
- [ ] **Grupos de permissao**: se feature nova, enum atualizado no backend e `permissionFeatures.ts` atualizado no frontend.

---

## 9) Convencoes de PR e Commit

### Commit

- Mensagens claras no estilo conventional commits (ex.: feat:, fix:, refactor:, docs:, test:, chore:).

### PR

Incluir obrigatoriamente:
- Contexto do problema.
- Escopo da solucao.
- Arquivos/areas impactadas.
- Evidencias de teste.
- Riscos e mitigacoes.
- Passo a passo rapido para validacao manual.

---

## 10) Referencias de Documentacao do Projeto

- docs/architecture.md
- docs/api.md
- docs/database.md
- docs/authentication.md
- docs/multi-tenancy.md
- docs/email.md
- docs/testing.md
- docs/deployment.md
- RELEASE.md

---

## 11) Estado Atual do Sistema (Funcionalidades Implementadas)

### 11.1 Envio de senhas por e-mail
- **Provider primario**: Resend (API key via RESEND_API_KEY, from via RESEND_FROM_EMAIL).
- **Fallback**: Brevo (BREVO_API_KEY, BREVO_SENDER_EMAIL, BREVO_SENDER_NAME).
- **Templates HTML profissionais**:
  - Template regular: cores do tenant (primary/secondary), logo grande circular com borda, info do consulente (nome, email, telefone), botao "Como chegar" via Google Maps, numero da senha em destaque.
  - Template patrocinador: paleta ouro/preto, mensagem de gratidao especial.
- **Seguranca**: HTML escaping em todos os campos de texto do usuario.
- Sem QR code. Sem botao de resgate. Nome do tenant no cabecalho do email.

### 11.2 Configuracao do Tenant (Admin)
- **Campos de branding**: nome, slug, logo (upload de imagem como BYTEA), cores (primary, secondary, font).
- **Endereco**: campo `endereco` em tenant_configs (migracao 011) — usado nos emails para o botao "Como chegar".
- **Feature flags**: habilitacao de walk-in, patrocinadores, etc.

### 11.3 Giras
- Campo "Local" removido do formulario de criacao/edicao e da tabela — endereco agora vem da config do tenant.

### 11.4 Porta (Visao da Porta)
- Gestao em tempo real da fila de atendimento com WebSocket.
- WebSocket URL usa `window.location.host` (sem porta hardcoded).
- nginx tem location regex para proxy WebSocket: `location ~ ^/api/v1/admin/giras/.+/door/ws$` com upgrade headers.
- Hook `useWebSocket` com reconexao automatica.
- Modais: AttendModal, WalkInModal.

### 11.5 Layout Admin (Sidebar)
- Header redesenhado: fundo gradiente com cores do tenant, logo circular 52px (ou avatar fallback com inicial), nome do terreiro como texto principal (ate 2 linhas), "Senhas Admin" como label secundario.
- Navegacao: Dashboard, Giras, Tickets, Porta, Usuarios, Analytics, Auditoria, Configuracoes.
- Item selecionado com gradiente do tenant.
- Footer: "Senhas v1.1 — Admin Edition".
- Responsivo: drawer temporario no mobile, permanente no desktop.
- Suporte a impersonacao (banner amarelo no topo).

### 11.6 Perfil do Usuario
- Upload de foto como BYTEA (armazenado no banco).
- Avatar exibido no AppBar e no sidebar.

### 11.7 Homepage Publica
- Favicon personalizado.
- Meta tags com Head do Next.js.

### 11.8 Cadeia de Migracoes Alembic
- 001 → 002 → 003 → 004 → 005 → 006 → 007 → 008 → 009a + 009b → 010 (merge) → 011 (tenant endereco) → ... → 015 → 016 (remove enterprise) → ... → 026 (stripe billing) → 027 (mensalidade mediun).
- Migracoes 009a e 009b foram merge na 010.
- Ultima migracao: 027_mensalidade_mediun.

### 11.10 Financeiro — Controle de Mensalidade de Mediuns (branch 002-financeiro-mensalidade)
- **Feature Premium**: disponivel apenas para tenants com plano PREMIUM (`mensalidade_mediun` flag via `subscription_info.py`).
- **Modelos**: `MensalidadeConfig` (valor_mensal, dia_vencimento, 1:1 tenant), `MensalidadePagamento` (UNIQUE mediun_id+mes, BYTEA comprovante), `MensalidadeStatus` enum (PENDENTE/PAGO/ISENTO).
- **Endpoints** (prefixo `/api/v1/admin/financeiro`): config GET/PUT, mensalidades GET/POST por mes, comprovante GET/DELETE, resumo GET (6 hist + 3 proj), relatorio POST enviar / GET download.
- **Regras de acesso**: leitura para OPERATOR+ADMIN, escrita (PUT config, POST pagamento, DELETE comprovante, POST relatorio) somente ADMIN/SUPER_ADMIN.
- **Comprovante**: BYTEA no banco, limite 5MB, tipos aceitos: jpeg/png/webp/pdf.
- **Relatorio**: email HTML gerado por `render_mensalidade_report()` com KPI cards + tabela inadimplentes.
- **Frontend**: `/admin/financeiro/mensalidades` (tabs Mediuns + Grafico com Recharts) e `/admin/financeiro/config`; sidebar com grupo Financeiro (gate `can('mensalidade_mediun')`).
- **Migration 027**: ENUM `mensalidade_status`, tabelas `mensalidade_configs` + `mensalidade_pagamentos`, coluna `mediuns.mensalidade_isento BOOLEAN DEFAULT false`.
- **Dependencia**: `python-dateutil` (usado em `mensalidade_repo.get_resumo` via `dateutil.relativedelta`).

### 11.11 Grupos de Permissão — Controle Fino de Acesso (branch 003-rbac-grupos-permissao)
- **Modelos**: `PermissionGroup` (1:M tenant), `GroupPermission` (grupo + feature + can_view/can_insert/can_edit/can_delete), `UserGroupMembership` (M:N associando users a grupos).
- **Regras de Acesso e Isolamento**: 
  - Apenas para operadores (`OPERATOR` role). Admins, Super Admins e sessões sob impersonation têm bypass total de grupos.
  - Multi-tenancy isolado estritamente no banco via queries com filtros `tenant_id` em repositório e serviços.
  - Consolidação acumulativa via lógica OR permissiva quando o operador pertence a múltiplos grupos.
  - Operadores sem grupo atribuído mantêm acesso total (retrocompatibilidade e convenção de onboarding).
- **Endpoints** (prefixo `/api/v1/admin/permission-groups`): CRUD completo de grupos, atribuição em massa de permissões (`/permissions`), associação/remoção de membros (`/members`), e retorno de permissões do usuário autenticado (`/me/permissions`).
- **Migração Alembic**: `b6d4a9b749d5_create_permission_groups.py` (tabelas e chaves estrangeiras com cascades).
- **Hooks e Providers**: `usePermissions` / `PermissionsProvider` gerenciando caching local (TTL 5 minutos), revalidação automática em focos de página ou eventos customizados de atualização de tenant.
- **Frontend / UI**:
  - `/admin/permission-groups`: listagem com filtros, alertas para operadores sem grupos (G1) e exclusão com proteção/força (G2).
  - `/admin/permission-groups/[id]`: detalhes do grupo, matriz de permissões (`PermissionMatrix`) com presets rápidos (G4), autocompletes de operadores e visualizador de permissões consolidadas (G3/G13).

### 11.9 Infraestrutura e Deploy
- Docker Compose com: postgres, redis, backend (FastAPI/Uvicorn), frontend (Next.js), nginx (reverse proxy + SSL).
- VPS: 76.13.231.19 (Hostinger), projeto em /opt/senhas.
- Dominio: girahub.com.br com SSL (Let's Encrypt).
- nginx: proxy reverso, terminacao SSL, WebSocket proxy para /door/ws.

**Deploy automatizado via GitHub Actions (`.github/workflows/deploy.yml`):**
1. Job `security-audit` (paralelo, nao-bloqueante): `pip-audit` + `npm audit --audit-level=high`.
2. Job `deploy` (SSH no VPS):
   - `pg_dump` backup antes de qualquer mudanca (mantém 10 backups em `/opt/senhas/backups/`).
   - `git pull` do repositorio.
   - Build das imagens com containers antigos AINDA rodando (zero-downtime durante build).
   - Migracao Alembic em container temporario (`--rm`).
   - Swap dos containers com `up -d`.
   - Health check com 5 retentativas antes de declarar sucesso.

**Deploy manual sem downtime (alternativa):**
```bash
cd /opt/senhas
# 1) Backup do banco:
docker exec senhas_postgres pg_dump -U senhas_user senhas_prod > /opt/senhas/backups/manual_$(date +%Y%m%d_%H%M%S).sql
# 2) Atualizar codigo:
git pull origin master
# 3) Build com containers antigos rodando:
docker compose -f docker-compose.prod.yml -f docker-compose.ssl.yml build backend frontend
# 4) Migracoes:
docker compose -f docker-compose.prod.yml -f docker-compose.ssl.yml run --rm backend alembic upgrade head
# 5) Swap:
docker compose -f docker-compose.prod.yml -f docker-compose.ssl.yml up -d backend frontend
```
NUNCA usar `up --build` direto — causa 503 prolongado durante o build.

**Rate limiter distribuído via Redis (desde 2026-06-27):**
- `REDIS_URL` adicionado ao `config.py` e ao `docker-compose.prod.yml` (backend environment).
- `limiter.py` usa `storage_uri=REDIS_URL` quando configurado; fallback in-memory em dev (REDIS_URL vazio).
- `limits[redis]` adicionado como dependencia em `pyproject.toml`.
- Hierarquia de roles refatorada em `dependencies.py`: `OPERATOR=0 < ADMIN=1 < SUPER_ADMIN=2` (dict `_ROLE_HIERARCHY`).
- PostgreSQL com `deploy.resources.limits.memory: 8G` no `docker-compose.prod.yml`.
- Backup retention aumentado de 10 para 30 no workflow CI.

**Monitoramento de erros — Sentry (desde 2026-06-27):**
- Backend: `sentry-sdk[fastapi]>=1.39.0` — inicializado em `main.py` quando `SENTRY_DSN` definido.
- Frontend: `@sentry/nextjs ^8` — configurado em `sentry.client.config.ts`, `sentry.server.config.ts`, `sentry.edge.config.ts`.
- DSNs ja configurados no VPS em `/opt/senhas/.env`.
- Variaveis: `SENTRY_DSN`, `NEXT_PUBLIC_SENTRY_DSN`, `SENTRY_ENVIRONMENT`, `SENTRY_TRACES_SAMPLE_RATE`.
- Em producao: `SENTRY_ENVIRONMENT=production`, `SENTRY_TRACES_SAMPLE_RATE=0.1`.
- MCP do Sentry disponivel via `.claude/settings.json` (url: `https://mcp.sentry.dev/mcp`).

---

## 12) Diretriz Final

Ao agir como agente de IA neste repositorio:
- Priorize seguranca e isolamento de tenant acima de velocidade.
- Nao suba segredos em nenhuma hipotese.
- Entregue mudancas testaveis, rastreaveis e bem documentadas.

---

## 13) Fluxo Operacional Padrao (SOP para Agentes)

Use este fluxo em toda implementacao, do inicio ao PR:

1. Entender o pedido e mapear impacto:
- Quais modulos serao tocados (backend, frontend, docs, migracoes)?
- Ha mudanca de contrato de API ou schema?

2. Levantar contexto minimo necessario:
- Ler arquivos diretamente relacionados.
- Identificar padroes existentes para manter consistencia.

3. Implementar em fatias pequenas:
- Aplicar mudancas objetivas e evitar refactor amplo sem necessidade.
- Preservar estilo e convencoes do repositorio.

4. Validar funcionalmente:
- Executar testes afetados (unitarios/integracao/componentes).
- Se mudanca ampla, rodar validacao adicional (build/typecheck/lint quando aplicavel).

5. Revisar seguranca e multi-tenancy:
- Conferir filtros de tenant_id em todas operacoes sensiveis.
- Verificar ausencia de segredos em codigo, docs e scripts.

6. Revisar diff final:
- Confirmar que nao ha alteracoes acidentais fora do escopo.
- Garantir mensagens de erro e logs sem dados sensiveis.

7. Preparar PR com contexto claro:
- Problema, solucao, impacto, testes, riscos, mitigacoes e passos de validacao.

### 13.1 Gate obrigatorio antes de push/PR

Antes de qualquer push:
- Confirmar que nao existem valores reais de senha, token, API key ou credenciais.
- Se houver qualquer suspeita de segredo no historico da branch, parar e sanitizar antes.

---

## 14) Template de PR para Agentes

Use este modelo ao abrir PR:

Titulo sugerido:
- tipo(escopo): resumo curto

Descricao:

### Contexto
- Problema de negocio/tecnico:
- Impacto atual:

### Solucao aplicada
- O que foi alterado:
- Decisoes tecnicas principais:
- Alternativas consideradas (se houver):

### Arquivos/areas impactadas
- Backend:
- Frontend:
- Banco/migracoes:
- Documentacao:

### Seguranca e multi-tenant
- Como tenant isolation foi preservado:
- Confirmacao de ausencia de segredos no diff/historico da branch:

### Evidencias de teste
- Testes executados:
- Resultado:
- Evidencias (logs/prints/saidas relevantes):

### Riscos e mitigacoes
- Riscos conhecidos:
- Mitigacoes aplicadas:

### Validacao manual rapida
1. Passo 1
2. Passo 2
3. Resultado esperado

### Checklist final
- [ ] Isolamento multi-tenant validado
- [ ] Sem segredos no repositorio
- [ ] Migracoes criadas (quando necessario)
- [ ] Testes relevantes passando
- [ ] Docs atualizadas (quando necessario)
- [ ] Grupos de permissao: backend com `require_group_permission` em todos os endpoints novos/alterados
- [ ] Grupos de permissao: frontend com `canGroup` bloqueando view e ocultando acoes sem permissao
- [ ] Grupos de permissao: nova feature adicionada ao enum e ao `permissionFeatures.ts` (se aplicavel)

---
> Source: [leonfpontes/Senhas](https://github.com/leonfpontes/Senhas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
