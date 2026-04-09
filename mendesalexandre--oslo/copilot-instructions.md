## oslo

> > Instruções para o agente Claude Code. Leia este arquivo antes de qualquer tarefa.

# CLAUDE.md — OSLO Sistema de Gestão Cartorária

> Instruções para o agente Claude Code. Leia este arquivo antes de qualquer tarefa.

---

## 1. Visão Geral

**OSLO** é um sistema SaaS de gestão cartorária com arquitetura multi-tenant.
Cada cartório (empresa cliente) possui seus próprios dados isolados por `empresa_id`.

| Camada     | Stack                                                  |
|------------|--------------------------------------------------------|
| Backend    | Laravel 12 · PHP 8.4+ · PostgreSQL · Redis             |
| Frontend   | Vue 3.4 · Quasar 2.18 · Pinia 3 · Axios               |
| Auth       | Laravel Sanctum (guard `web`, HttpOnly cookies, stateful) |
| PDF        | mPDF 8 + wkhtmltopdf (Snappy)                          |
| Fila       | Redis (queues + sessions + cache)                      |
| Assinatura | Lacuna/RestPKI                                         |

---

## 2. Estrutura de Repositórios

```
/home/alexandre/code/OSLO/          → git remote: github.com:mendesalexandre/OSLO.git
├── backend/                         → código Laravel (API)
├── frontend/                        → código Vue/Quasar (app)
├── docs/                            → documentação do projeto
├── CLAUDE.md
└── MULTI_TENANCY.md
```

**Monorepo único** — todos os commits e push vão para `github.com:mendesalexandre/OSLO.git`.
Os repositórios antigos `OSLO-api` e `OSLO-app` no GitHub estão preservados mas não são mais usados.

---

## 3. Comandos Essenciais

### Backend

```bash
# Desenvolvimento
php artisan serve --port=8000        # ou Octane: php artisan octane:start

# Banco de dados
php artisan migrate
php artisan migrate:fresh --seed     # recria tudo

# Seeders individuais (ordem importa!)
php artisan db:seed --class=TabelaCustaModeloSeeder
php artisan db:seed --class=TabelaCustaImpostoSeeder   # antes dos Atos!
php artisan db:seed --class=TabelaCustaAtoSeeder

# Auditoria
php artisan auditoria:aplicar        # aplica triggers em todas as tabelas
php artisan auditoria:migration      # gera migration de trigger para uma tabela

# Email manual (dev)
php artisan usuario:ativar-email     # ativa último usuário criado
php artisan usuario:ativar-email usuario@exemplo.com

# Cache
php artisan config:clear && php artisan cache:clear   # executar antes de migrate!

# Testes
php artisan test
./vendor/bin/pest
```

### Frontend

```bash
quasar dev      # desenvolvimento (porta 9000)
quasar build    # build produção
```

---

## 4. Convenções de Banco de Dados

### Nomes de colunas
- Timestamps customizados: `data_cadastro`, `data_alteracao`, `data_exclusao`
- Tabela de usuários: `usuario` (não `users`)
- Senha do usuário: `senha` (não `password`) → `User::getAuthPassword()` faz o override
- IDs de auditoria: `criador_id`, `alterador_id`, `excluidor_id` → FK para `usuario`
- UUIDs auto-gerados no `boot()` do model

### Traits obrigatórios por modelo
| Situação | Trait |
|---|---|
| Model com dados de empresa | `PertenceEmpresa` (adiciona EmpresaScope automático) |
| Model com auditoria via trigger | `Auditavel` |
| Model com upload de arquivos | `Arquivavel` (morphMany arquivos) |
| Model com soft delete | `SoftDeletes` + timestamps customizados |

### Timestamps no Model
```php
const CREATED_AT = 'data_cadastro';
const UPDATED_AT = 'data_alteracao';
const DELETED_AT = 'data_exclusao';   // apenas com SoftDeletes
```

### Trait collision (SoftDeletes + TimestampsPortugues)
Ambas definem `getDeletedAtColumn()`. Resolver com `insteadof`:
```php
use SoftDeletes, TimestampsPortugues {
    TimestampsPortugues::getDeletedAtColumn insteadof SoftDeletes;
}
```

### Pivot tables com timestamps customizados
Se um model tem `CREATED_AT`/`UPDATED_AT` customizados, a pivot **herda** esses nomes.
Sempre especificar explicitamente em `BelongsToMany`:
```php
->withTimestamps('created_at', 'updated_at')
```

### Migrations PostgreSQL
- Para dropar tabelas com FK: `DB::statement("DROP TABLE IF EXISTS tabela CASCADE")`
- `morphs()` já cria índice — não adicionar manualmente ou causa erro de duplicidade
- Migrations de auditoria (triggers) devem ter timestamp **depois** da tabela-alvo

---

## 5. Padrões de API

### Response format

Todos os controllers **novos** usam o trait `RespostaApi` (português):
```php
use App\Traits\RespostaApi;

$this->sucesso($dados, 'mensagem');           // 200
$this->criado($dados, 'mensagem');            // 201
$this->erro('mensagem', 422, $erros);         // 4xx
$this->naoEncontrado('mensagem');             // 404
$this->sucessoPaginado($paginador);           // 200 paginado
```

Controllers **legados** usam `ApiResponseTrait` (inglês — não usar em código novo).

### Formato de resposta
```json
{ "sucesso": true, "mensagem": "...", "dados": { ... } }
```

### Rotas protegidas
```php
// Todas as rotas autenticadas passam por:
Route::middleware(['auth:api', 'email.verificado', 'trial'])->group(function () {
    // rotas aqui
});
```

### Permissões nas rotas
```php
->middleware('permissao:NOME_PERMISSAO')           // 1 permissão
->middleware('permissao:PERM_A,PERM_B')            // qualquer uma (OR)
```

---

## 6. Sistema de Permissões (RBAC)

### Módulos e permissões (~138 no total)

| Módulo | Prefixo | Permissões |
|---|---|---|
| Protocolo | `PROTOCOLO_` | LISTAR, CRIAR, VISUALIZAR, EDITAR, CANCELAR, PAGAR, ESTORNAR, PAGAMENTO_EXCLUIR, ISENTAR |
| Contrato | `CONTRATO_` | LISTAR, CRIAR, VISUALIZAR, EDITAR, CONCLUIR, CANCELAR |
| Recibo | `RECIBO_` | LISTAR, VISUALIZAR, GERAR |
| Arquivo | `ARQUIVO_` | LISTAR, VISUALIZAR, EXCLUIR |
| Ato | `ATO_` | LISTAR, CRIAR, VISUALIZAR, EDITAR, EXCLUIR |
| Financeiro | `FORMA_PAGAMENTO_`, `MEIO_PAGAMENTO_`, `TRANSACAO_` | LISTAR, CRIAR, VISUALIZAR, EDITAR, EXCLUIR |
| Caixa | `CAIXA_`, `CAIXA_MOVIMENTO_`, `CAIXA_OPERACAO_` | LISTAR, ABRIR, FECHAR, CONFERIR, SANGRIA, etc. |
| Administração | `DOMINIO_`, `NATUREZA_`, `ESTADO_`, `CIDADE_`, etc. | LISTAR, CRIAR, VISUALIZAR, EDITAR, EXCLUIR |

### Grupos padrão
- **Administrador** — bypass total (método `eAdmin()`)
- **Registrador** — operacional completo
- **Atendente** — protocolo e contrato
- **Caixa** — financeiro
- **Consulta** — somente leitura

### Frontend
```javascript
// Composable
const { temPermissao } = usePermissao()
temPermissao('PROTOCOLO_CRIAR')

// Diretiva
v-permissao="'PROTOCOLO_CRIAR'"

// Menu
{ permissao: ['PROTOCOLO_LISTAR'] }
```

---

## 7. Multi-Tenancy

### Como funciona
- Toda tabela de dados tem `empresa_id` (FK → empresa)
- `PertenceEmpresa` trait aplica `EmpresaScope` automaticamente — **nunca** precisa adicionar `where empresa_id` manualmente
- `empresa_id` é preenchido automaticamente no `create()`

### Tabelas SEM empresa_id (dados globais/sistema)
`plano`, `natureza`, `ato`, `ato_faixa`, `forma_pagamento`, `meio_pagamento`, `etapa`, `tabela_custa`, `tabela_custa_ato`, `tabela_custa_imposto`, `estado`, `cidade`

### Query admin (todas empresas)
```php
Protocolo::withoutGlobalScope(EmpresaScope::class)->get();
// ou
Protocolo::semEmpresa()->get();
```

---

## 8. Auditoria (PostgreSQL Triggers)

- Schema: `auditoria`, tabela: `auditoria.registro`
- Trigger `fn_registrar_auditoria()` captura INSERT/UPDATE/DELETE
- Middleware `AuditoriaContexto` injeta `SET LOCAL app.usuario_id` / `ip_address` / `user_agent`
- **Não confundir** com `AuditableTrait` (Eloquent-based, legado em Dominio/Natureza/Sequencia)

```bash
# Aplicar trigger em nova tabela
php artisan auditoria:aplicar nome_tabela
```

---

## 9. Workflow de Protocolo (Etapas)

```
Atendimento → Distribuição → Análise → Registro → Concluído
                                    ↘ Exigência
                                    ↘ Cancelado
```

- Transições definidas em `Protocolo::mapaTransicoes()`
- Service: `ProtocoloEtapaService::avancar()` e `voltar()`
- Cada transição gera registro em `protocolo_andamento`
- Rotas: `POST /protocolo/{id}/etapa` e `POST /protocolo/{id}/etapa/voltar`

---

## 10. Tabela de Custa / Cálculo de Emolumentos

### Tipos de cálculo (`TipoCalculoAtoEnum`)
- `FIXO` — `valor_servico` × quantidade
- `FAIXA_PROGRESSIVA` — `valor_servico` + `ceil((base - valor_inicio_incremento) / valor_faixa)` × `valor_acrescimo` (teto: `valor_maximo`)
- `GRATUITO` — zero

### Impostos MT 2025 (ordem de aplicação)

| Ordem | Código | Tipo | Valor | Base | Deduz |
|---|---|---|---|---|---|
| 1 | `registro_civil` | fixo | R$ 8,20 | — | sim |
| 2 | `funajuris` | percentual | 20% | emolumento_liquido pós-RC | sim |
| 3 | `funamp` | percentual | 5% | emolumento_liquido pós-RC+FUNAJURIS | sim |
| 4 | `issqn` | percentual | 5% | emolumento_liquido pós-RC+FUNAJURIS+FUNAMP | não |

**Total cliente** = emolumento_bruto + ISSQN

### Endpoint de cálculo
```
POST /api/tabela-custa-ato/{ato}/calcular
Body: { "base_calculo": 50000.00, "quantidade": 1 }
Permissão: TABELA_CUSTA_LISTAR
```

### Seeders (executar nesta ordem)
```bash
php artisan db:seed --class=TabelaCustaModeloSeeder    # cria a tabela de custa
php artisan db:seed --class=TabelaCustaImpostoSeeder   # cria os impostos
php artisan db:seed --class=TabelaCustaAtoSeeder       # importa atos do JSON
```

JSON dos atos: `database/seeders/json/tabelacusta2025.json`

**Gotcha do seeder de atos:**
- `tabela_custa` não tem `estado_id` — lookup por `nome` + `ano`
- JSON tem campos `permitir_funajuris/registro_civil/iss` que **não existem na tabela** — são mapeados para a pivot `tabela_custa_ato_imposto`
- FUNAMP segue o mesmo flag do FUNAJURIS (adicionado depois)

---

## 11. Upload de Arquivos

- Tabela polymorphic `arquivo` com `arquivavel_type` / `arquivavel_id`
- Trait `Arquivavel` fornece `arquivos()` morphMany
- Storage: `storage/app/protocolo-pagamentos/{protocolo_id}/{uuid}.ext`
- Limites: 5MB, aceita JPG/PNG/PDF
- Download: `GET /arquivo/{uuid}/download?token=JWT`

---

## 12. Auth Flow

### Backend
- Guard: `api` (driver JWT), config em `config/auth.php` e `config/jwt.php`
- Token blacklist via Redis
- `User::getAuthPassword()` retorna `$this->senha` (coluna customizada)

### Frontend
- Token em `localStorage('access_token')`
- Axios interceptor em `src/boot/axios.js`:
  - 401 → tenta refresh automático + fila de requests pendentes
  - 403 `TRIAL_EXPIRADO` → notifica e redireciona para `/planos`
  - 403 `EMAIL_NAO_VERIFICADO` → notifica
- Auth store em `src/stores/auth/index.js` — persiste permissões, módulos, grupos

---

## 13. Cadastro Multi-Tenant (Público)

```
POST /api/publica/cadastro              → cria empresa + usuário admin + trial 7 dias
GET  /api/publica/verificar-email/{token}
POST /api/publica/reenviar-verificacao
GET  /api/publica/plano                 → planos públicos
```

### Status da empresa
`trial` → `ativo` → `read_only` (trial expirado) → `suspenso` → `cancelado`

Middleware `VerificarTrial` bloqueia POST/PUT/PATCH/DELETE quando `read_only`.

---

## 14. Localização de Arquivos Chave

### Backend
| O quê | Onde |
|---|---|
| Rotas | `routes/api.php` |
| Middleware aliases | `bootstrap/app.php` |
| Auth guard | `config/auth.php` |
| JWT config | `config/jwt.php` |
| User model | `app/Models/User.php` |
| Auth controller | `app/Http/Controllers/AuthController.php` |
| Permissões (RBAC) | `app/Http/Middleware/ChecarPermissao.php` |
| Cálculo emolumentos | `app/Services/TabelaCustaService.php` |
| Auditoria trigger | `app/Console/Commands/AuditoriaAplicarCommand.php` |
| Email templates | `resources/views/emails/` |

### Frontend
| O quê | Onde |
|---|---|
| Axios + interceptors | `src/boot/axios.js` |
| Auth store | `src/stores/auth/index.js` |
| Menu config | `src/config/menu.js` |
| Permissão composable | `src/composables/usePermissao.js` |
| Layout principal | `src/layouts/MainLayout.vue` |
| Rotas | `src/router/routes.js` |

---

## 15. Regras de Desenvolvimento

### Sempre fazer
- Usar `RespostaApi` trait em controllers novos (português)
- Usar `PertenceEmpresa` em qualquer model que armazene dados por empresa
- Usar `updateOrCreate` em seeders (idempotente)
- Aplicar `php artisan config:clear` antes de rodar migrations
- Especificar `->withTimestamps('created_at', 'updated_at')` em BelongsToMany se o model pai tem timestamps customizados
- **Sempre usar a store Pinia para chamadas de API** — nunca chamar `api` diretamente em pages ou components
- **Separar dialogs/modais em componentes próprios** (ex: `ModalTabelaCustaAto.vue`) e usar `<modal v-model>` global
- **Consultas/buscas (telas de consulta unificada, busca avançada, etc.) sempre devem ser modais** usando `<modal v-model="aberto">` — nunca criar página standalone para consulta
- **Controllers com UUID**: nunca usar route model binding (`TabelaCustaAto $ato`) para rotas que recebem UUID — usar `string $uuid` + lookup manual `where('uuid', $uuid)->first()`

### Nunca fazer
- Criar `$guarded = []` sem `$fillable` explícito em models com dados sensíveis
- Adicionar índice manualmente após `$table->morphs()` (já cria automaticamente)
- Commitar `.env` ou credenciais
- Usar `ApiResponseTrait` (inglês) em código novo — usar `RespostaApi`
- Fazer push para `main` sem criar tag de versão quando houver feature completa
- Chamar `api.get/post/put/delete` diretamente em `.vue` — **sempre via store**
- Usar `q-dialog` inline em pages — **sempre criar componente separado** com `<modal>`
- Criar página separada para consultas — **sempre usar modal** (`<modal v-model>`)

### Padrão de tags Git
```
v{ANO}.{MES}.{DIA}-{descricao-curta}
# Exemplos:
v2026.02.22-tabela-custa
v2026.02.22-sidebar-expand
```

---

## 16. Enums disponíveis

| Enum | Valores |
|---|---|
| `TipoCalculoAtoEnum` | FIXO, FAIXA_PROGRESSIVA, GRATUITO |
| `TransacaoNaturezaEnum` | — |
| `TransacaoStatusEnum` | — |
| `TransacaoTipoEnum` | — |
| `CaixaMovimentoStatusEnum` | — |
| `CaixaOperacaoTipoEnum` | — |
| `CategoriaTipoEnum` | — |

---

## 17. Dependências Externas

| Serviço | Integração | Status |
|---|---|---|
| Lacuna/RestPKI | API Key header | ativo |
| DOI/RFB (Receita) | Cookie session | ativo |
| BrasilAPI (CNPJ) | HTTP GET (gratuito) | ativo |
| CEI/MT | não implementado | — |
| RTDPJ | não implementado | — |
| NEXTYR | não implementado | — |
| ONR | **REMOVIDO** (2026-02-07) | — |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mendesalexandre)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mendesalexandre)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
