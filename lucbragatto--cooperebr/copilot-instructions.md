## cooperebr

> **Inegociável** — toda sessão Code (Claude Code CLI) **abre** apresentando

# Instruções permanentes — Claude Code no CoopereBR

## Ritual de abertura e fechamento de sessão

**Inegociável** — toda sessão Code (Claude Code CLI) **abre** apresentando
"Onde paramos + Pendências" e **fecha** atualizando o mesmo registro vivo.

Formato fixo em `~/.claude/projects/C--Users-Luciano-cooperebr/memory/ritual_abertura_fechamento.md`.

Estado vivo fica em `docs/CONTROLE-EXECUCAO.md` (seção **"ONDE PARAMOS"**).

Aplica:
- Toda sessão Code, mesmo se for "continuação" ou "mesmo dia"
- claude.ai web por reflexo (Luciano cola `CONTROLE-EXECUCAO.md` ao abrir)

**Origem:** Luciano em 2026-05-02. Necessidade de não perder contexto entre
sessões e ter pendências sempre visíveis.

## Fechamento de sessão — REGRA INEGOCIÁVEL BILATERAL (13/05/2026)

Reforço da regra de ritual de fechamento. Aplicação bilateral
claude.ai + Code.

### Ao final de TODA sessão (claude.ai OU Code)

Executar 3 itens obrigatórios antes de standby:

1. **Doc-sessão em `docs/sessoes/YYYY-MM-DD-[escopo].md`** contendo:
   - TL;DR 3-5 linhas legível pra leigo
   - Lista de entregas + SHAs dos commits
   - Lista de débitos novos catalogados
   - Lista de débitos resolvidos
   - Lista de bugs descobertos durante validação
   - Pendências abertas pra próxima sessão
   - Decisões catalogadas
   - Próximo passo único e claro

2. **`docs/CONTROLE-EXECUCAO.md` atualizado:**
   - Seção "Última sessão" reescrita com resumo
   - Frase de retomada COMANDANTE atualizada (formato copy-paste pro
     Code, NÃO descritiva tipo "próxima sessão fará X")

3. **`docs/diagramas/jornada-membro.html` atualizado (Sugestão #6 da
   memória persistente claude.ai — ritual_atualizacao_jornada_diaria):**
   - Se sessão modificou estado do projeto (novo cooperado, débito,
     engine, sprint), regenerar HTML versão nova (bump v1.X)
   - Refletir mudanças: cores do diagrama, tabela de status, pontos
     críticos, próximos passos, changelog

4. **Commit + push obrigatórios**
   - Mensagem: `docs(sessao): fechamento DD/MM - [resumo curto]`
   - Push obrigatório — não deixar commits locais

### Bilateral — lembranças mútuas

**Se claude.ai esquecer (só empacotar commits de código sem fechamento
documental):**

Code, no ÚLTIMO turno antes de entrar em standby de qualquer prompt,
DEVE verificar:
- Existe `docs/sessoes/YYYY-MM-DD-*.md` correspondente ao dia atual
  refletindo o que rodou nesta sessão?
- `CONTROLE-EXECUCAO.md` seção "Última sessão" reflete realidade
  recente?
- Frase de retomada atualizada?
- Há commits locais sem push (`git log origin/main..HEAD`)?

Se algum NÃO: ANTES de standby, perguntar ao Luciano:

> "Sessão sem fechamento documentado detectado. Posso criar
> docs/sessoes/YYYY-MM-DD-[escopo].md + atualizar CONTROLE-EXECUCAO +
> push? (regra inegociável 13/05/2026)"

**Se Code esquecer (próxima sessão):**

Abertura de próxima sessão claude.ai deve checar:
- Última doc em `docs/sessoes/` é do dia anterior?
- Há gap entre data da última doc-sessão e commits recentes (`git log
  --since=...`)?

Se gap detectado: claude.ai puxa contexto via git log + cria
fechamento retroativo ANTES de qualquer trabalho novo.

### Quando dispensar

**Nunca.** Mesmo sessões curtas (15min) ou só leitura merecem
registro. Sessão de leitura gera doc curta tipo:
"Sessão leitura DD/MM 15min — consultei docs/X, decidi Y, próximo
passo Z."

### Por que essa regra existe

13/05/2026 — sessão maratona Sub-Fase A canário + D-48 + D-50..D-55
acumulou 11+ commits sem doc-sessão consolidada. Luciano apontou.
Memória persistente claude.ai estava atualizada (lado claude.ai),
mas repo do projeto estava sem rastreabilidade narrativa do que
aconteceu. Sem regra forte, débitos se acumulam invisíveis.

Detalhamento completo em memória persistente claude.ai
`~/.claude/projects/C--Users-Luciano-cooperebr/memory/regra_fechamento_sessao_inegociavel.md`.

## Contatos de teste — REGRA INEGOCIÁVEL (14/05/2026)

Para QUALQUER teste que dispare comunicação real (SMTP, WhatsApp,
Asaas boleto/PIX, notificações) OU movimentação financeira real:

**Substituir IMPRETERIVELMENTE contatos do cooperado por:**
- Telefone/WhatsApp: `27981341348`
- Email: `lucbragatto@gmail.com`

**CPF/CNPJ mantém** (Asaas valida CPF da pessoa física/jurídica real).

**Aplicação obrigatória:**
- Testes de envio Asaas (boleto/PIX em produção)
- Testes WhatsApp (notificações, lembretes, confirmações)
- Testes email (faturas, comunicações, lembretes)
- Sub-canários (CAROLINA, AMAGES, qualquer outro)
- Desativação de `ambienteTeste=true` em piloto
- Testes E2E que envolvam SMTP/IMAP/WA/Asaas

**Quando NÃO aplicar:**
- Cadastros via wizard normal (mantém dados reais + `ambienteTeste=true`)
- Testes sintéticos (whitelist LGPD já bloqueia)
- Operações read-only no banco

**Origem:** Luciano em 14/05/2026 — "todos os dados são extraídos do
sistema que temos, embora sejam reais, não podemos contatá-los, use
sempre e impreterivelmente meu celular e meu email para testes".

**Bilateral:** Code verifica antes de qualquer disparo real e pergunta
se contatos devem ser substituídos. Claude.ai catalogou em memória
persistente `regra_contato_teste_impreterivel.md`.

**Refinamento email com `+suffix` (14/05/2026):**

Luciano é cooperado real CoopereBR com email `lucbragatto@gmail.com`. Por
unique constraint Prisma, nenhum outro cooperado pode usar o mesmo email
base.

Solução: padrão Gmail `+suffix` (RFC-compliant). Aliases:
- `lucbragatto+carolina@gmail.com`
- `lucbragatto+diego@gmail.com`
- `lucbragatto+almir@gmail.com`
- `lucbragatto+theomax@gmail.com`
- `lucbragatto+amages@gmail.com`
- `lucbragatto+marcio@gmail.com`
- Futuros: `lucbragatto+<novo>@gmail.com`

Whitelist em `backend/src/common/safety/whitelist-teste.ts` já lista os 6
aliases canônicos. Gmail roteia tudo pra mesma caixa.

## Disciplina de validação prévia (Decisões 14, 15, 20)

**Regra inegociável** — em três granularidades cumulativas:

**Antes de cada resposta** (Decisão 20, 02/05/2026): verificar docs + código +
sessões anteriores sobre o tema. NÃO responder "de cabeça".

**Antes de propor sprint** (Decisão 20): verificar pilha existente
(`PLANO-ATE-PRODUCAO.md`) + sub-sprints (Decisão 18) + débitos
(`debitos-tecnicos.md`) + sugestões (`sugestoes_pendentes.md`). Se conflito:
**reportar + perguntar** antes de propor.

**Antes de retomar sessão** (Decisão 15, 01/05/2026): ler `CONTROLE-EXECUCAO.md` +
cruzar `git log -20` + verificar memória persistente.

**Antes de trabalho novo** (Decisão 14, 30/04/2026): cruzar `docs/` + código +
schema + git antes de propor solução.

Detalhes em `~/.claude/projects/C--Users-Luciano-cooperebr/memory/regra_validacao_previa_e_retomada.md`.

Aplica-se a Code, claude.ai e qualquer agente futuro.

**Por quê:** sessões que pulam essa etapa produzem retrabalho, conflitos de numeração,
órfãos esquecidos, divergência entre documentação/código/banco/operação. A coerência
sistêmica depende dessa disciplina.

**Violações documentadas:**
- **30/04 noite:** claude.ai propôs nova numeração de sprints sem validar com a antiga
  → 5 colisões + 6 órfãos (commit `1be9b34`).
- **02/05 (manhã+tarde):** múltiplas violações dentro da mesma sessão (specs CooperToken
  omitidos, Planos comerciais omitidos, respostas "de cabeça"). Decisão 20 nasceu daqui.

**Origem:** sessões claude.ai 30/04 (Decisão 14), 01/05 (Decisão 15), 02/05 (Decisão 20).

## Disciplina de análise — modelo canônico primeiro (16/06/2026)

Antes de analisar/propor qualquer coisa que toque dinheiro, contabilidade, modelo de dados ou regra econômica, derivar PRIMEIRO o modelo canônico ("como deveria ser") com as 4 lentes — contador/financista, engenheiro de sistemas, DBA sênior, negócios — e medir o código contra ele. Desvios viram débitos com o carimbo certo. Não rotular gaps reativamente. Artefato vivo do token: `docs/FUNDACAO-COOPERTOKEN-MODELO-CANONICO.md`.

## Cooperados institucionais — SALVAGUARDA (Fatia F-G1, 05/06/2026)

Cada cooperativa pode ter um **cooperado fantasma** com nome
`"{Nome da cooperativa} — Institucional"` e email
`institucional+<cooperativaId>@sisgd.invalid` (RFC 2606 — domínio
reservado, nunca roteável).

São **registros de sistema** usados como `cooperadoIndicadorId` em
convites de indicação institucionais (criados pelo admin da cooperativa
via `POST /convite-indicacao/admin` sem `indicadorId` específico — Fatia
F-G1 do Circuito CooperToken).

**REGRAS INEGOCIÁVEIS:**
- **NUNCA deletar** cooperados institucionais. Não são "dado de teste"
  mesmo com aparência sintética — são registros de sistema.
- **Qualquer rotina automática de limpeza de dados de teste DEVE excluir
  explicitamente** os emails `LIKE 'institucional+%@sisgd.invalid'`.
- O service `CooperadoInstitucionalService.garantirInstitucional` é
  idempotente — chame on-demand, nunca duplica.
- Quando o convite institucional vira `Indicacao` + 1ª fatura paga:
  `processarPrimeiraFaturaPaga` detecta via `ehInstitucional()` e
  **pula a emissão de BeneficioIndicacao + tokens** (não há referrer
  real pra premiar). A `Indicacao` é criada (rastreabilidade preservada).

## Antes de qualquer tarefa, SEMPRE ler primeiro (nesta ordem)

1. `docs/COOPEREBR-ALINHAMENTO.md` — estado consolidado do projeto
2. `docs/MAPA-INTEGRIDADE-SISTEMA.md` — diagnóstico ponta a ponta dos 10 fluxos
3. `docs/sessoes/` — sessões recentes (os 3 arquivos mais novos)
4. `git log -5 --oneline` — últimos commits

## Mapa de Integridade (documento vivo)

`docs/MAPA-INTEGRIDADE-SISTEMA.md` é atualizado ao final de cada sprint.
Após fechar sprint, atualizar a matriz executiva (% pronto, gaps resolvidos).
Não criar versão nova com data — sobrescrever o mesmo arquivo.
Se precisar do histórico, git log mostra as versões anteriores.

Esses 3 em ordem garantem contexto completo em 5 minutos.

## Sobre o projeto em 5 linhas

**SISGD** é a plataforma SaaS multi-tenant de Geração Distribuída.
Dono: Luciano (não programa).
Parceiros (cooperativas/consórcios/associações/condomínios) pagam Luciano pelo uso do sistema via FaturaSaas.
Membros dos parceiros pagam seus parceiros (não pagam Luciano).
**CoopereBR é UM parceiro entre vários possíveis, NÃO o dono do sistema.**

Detalhes em `docs/COOPEREBR-ALINHAMENTO.md` e `docs/PRODUTO.md` (visão humana atual).
Histórico: `docs/historico/SISGD-VISAO-COMPLETA-2026-04-26.md`.

## Vocabulário multi-tipo (regra dura)

SISGD atende 4 tipos de parceiro, cada um com nome próprio pra "membro":

| `tipoParceiro` (enum) | Membro singular | Membro plural |
|---|---|---|
| COOPERATIVA | Cooperado | Cooperados |
| CONSORCIO | Consorciado | Consorciados |
| ASSOCIACAO | Associado | Associados |
| CONDOMINIO | Condômino | Condôminos |

**Regras:**
- Tabela legado se chama `Cooperativa` mas representa **qualquer parceiro**. Não renomear.
- UI/templates **nunca** devem hardcodar "Cooperado" — usar hook `useTipoParceiro()` (`web/hooks/useTipoParceiro.ts`) que retorna `{tipoMembro, tipoMembroPlural}` baseado no `tipoParceiro` da cooperativa logada.
- Hangar Academia, AESMP, ASSEJUFES são **membros PJ da CoopereBR (cooperados)** — não são parceiros do SISGD.
- Hook já adotado em 21 telas. Ainda há ~50 telas + 73 exceptions backend com termo hardcoded — débito P2 registrado em `docs/debitos-tecnicos.md` (commit `91652ae`). **Bloqueia onboarding produção de Consórcio/Associação/Condomínio**, não bloqueia desenvolvimento.

## Arquitetura de rotas (Sprint Higiene 14/06/2026 — D-1171 fechado)

Após Sprint Higiene de Rotas (Decisões D1+D2+D3+D4), o frontend tem **5 áreas
canônicas por perfil** (mapeadas em `hooks/useContexto.ts:rotaPorContexto`):

| Contexto JWT | Rota home | Quem usa |
|---|---|---|
| `super_admin` | `/dashboard` | Dono do SISGD — operação multi-tenant, "Gestão Global" visível |
| `admin_parceiro` | `/dashboard` | Admin do tenant — MESMA área do super-admin, mas sem "Gestão Global"; sidebar mostra "Painel Administrativo — {nomeCooperativa}" |
| `cooperado` | `/portal` | Cooperado regular — fatura, UCs, tokens, indicações |
| `empresa_conveniada` | `/conveniada` | Empresa cooperada PJ — distribuir tokens em lote aos funcionários |
| `proprietario_usina` | `/proprietario` | Dono de usina — produção, despesas, repasses |
| `admin_agregador` | `/agregador` | Agregador externo |
| `cooperado` + `ehEstabelecimento=true` | `/estabelecimento` | Cooperado-estabelecimento do Clube — recebe pagamento em CooperToken, valida resgates. Card de entrada visível em `/portal/page.tsx` |

**Decisões consolidadas:**
- D1: `/parceiro/*` DESCONTINUADO (todas as ~25 telas foram convergidas em `/dashboard/*`; 33 redirects 301 `permanent:true` em `next.config.ts` preservam deep-links legados).
- D2: `/estabelecimento/*` é área NOVA (Sprint Higiene Bloco B), guard de `ehEstabelecimento` no layout reusa contexto `cooperado` (sem novo contexto JWT v1).
- D3: ADMIN do parceiro vê `h1 "Painel Administrativo"` + `p "{nomeCooperativa}"` em `/dashboard/layout.tsx`. Super-admin vendo um tenant específico em `/dashboard/parceiros/[id]` vê `"Painel do Tenant — {nome}"`.
- D4: `/portal/comprar-tokens` permanece canônica (PF+PJ); link adicional no `/conveniada/*` aponta pra lá quando aplicável (sem duplicação de rota).

**Glossário travado (memória `modelo_portais_e_colisao_parceiro_14_06_2026.md`):**
- **PARCEIRO** = tenant (a cooperativa/consórcio/associação/condomínio cadastrado no SISGD).
- **ADMIN DO PARCEIRO** = pessoa que opera o tenant (perfil `ADMIN` no JWT).
- **ESTABELECIMENTO** = loja/serviço que aceita CooperToken no Clube (cooperado PF/PJ com `ehEstabelecimento=true`).

## Convenções de código

- Multi-tenant: toda query Prisma filtra por `cooperativaId`
- `npx prisma db push` em dev (nunca migrate)
- PowerShell: `;` em vez de `&&`
- Commits em português, pequenos, descritivos
- Valores monetários: `Math.round(x * 100) / 100`

## Como trabalhar com Luciano

- Luciano NÃO programa
- Explicar decisões em linguagem humana, sem jargão técnico
- Decisões técnicas puras (estrutura, libs, organização): decide você, comunica motivo
- Decisões de produto (regra de negócio, fluxo de usuário): pergunta antes de executar

## Quando Luciano pedir conteúdo de arquivo

- Execute `Get-Content <path>` e cole o output LITERAL
- NÃO resumir, NÃO interpretar
- Se arquivo > 500 linhas, avisar antes e perguntar se quer em partes

## Sprint atual

Sprint 13a P0 e Dia 1 fechados (28/04/2026). Painel SISGD `/dashboard/super-admin` operacional.

**Próximo: Sprint 13a Dia 2** — lista de parceiros enriquecida + filtros + smoke test.

Sprint 13 foi dividido em 3 fatias entregáveis (não monolítico):
- **13a** (em andamento) — Painel super-admin (Dia 1 ✅, Dia 2 e 3 pendentes)
- **13b** — AuditLog ativo (interceptor) + Impersonate completo
- **13c** — Edição de plano SaaS pelo painel + suspensão de parceiro

## Módulo Clube + CooperToken

Especificação em `docs/especificacao-clube-cooper-token.md`.
Sprint 8 implementa MVP. Sprint 9 faz rede interna. Sprint 10+ rede
aberta (requer consulta advogado antes).

Princípio: token = desconto diferido. kWh constante. Cooperado escolhe
Desconto (imediato) ou Clube (acumular tokens).

Ferramentas configuráveis (ativáveis por parceiro):
- Expiração de tokens (prazo em meses)
- Desvalorização temporal (período graça + taxa + piso)

Regra de ouro: comunicação transparente ao cooperado. Curvas
sempre visíveis no portal. WhatsApp notifica antes de eventos.

Antes de implementar Sprint 8, investigar /cadastro público —
pode já ter partes do Clube.

### Circuito CooperToken + Convênio — desenho consolidado (04/06/2026)

Plano de finalização completo em **`docs/especificacao-circuito-cooper-token-convenio.md`**
(diagrama visual em `Downloads/circuito-cooper-token-completo.html`). Decisões travadas:

- **COOPERADO-ONLY:** só cooperado (PF ou PJ) participa do Clube ou faz convênio (já no
  estatuto) → tudo é ato cooperativo (Art. 79). "Parceiro do Clube" = cooperado PF/PJ
  (Sprint Higiene 14/06: agora chamado **"estabelecimento"** na UI, área dedicada
  `/estabelecimento/*`; **`/parceiro/*` foi descontinuado** e convergido em `/dashboard/*` —
  ver Sprint Higiene de Rotas abaixo).
- **Dois rios:** ENERGIA→SOBRA (dinheiro, proporcional à energia, mensal) × TOKEN→
  BENEFÍCIO (circuito fechado; abate fatura/parceiros; sai por **resgate**). Sobra
  acompanha energia, NUNCA token.
- **Token = voucher de circuito fechado**, cooperativa = emissora única. Saída em R$:
  parceiro = **resgate/liquidação** (recibo, sem NF de venda); cooperado = **sobra**.
  **Nunca "recompra".** Coop tributa só spread/queima; valor cheio é trânsito.
- **Convênio = 2 pontas (energia + token), fatura única SEGREGADA** (Art. 87 obrigatório).
- 3 pareceres `cooperebr-analista-conformidade` + memória `decisao_modelo_token_voucher_sobra_resgate_2026_06_04.md`.
- ⚠️ STF Tema 536 (isenção ato cooperativo) em julgamento — monitorar.
- Após build+teste do circuito → **Módulo Contabilidade Tributária Segregada**
  (atualizado com energia×token×quebra/queima).

## Gateways de pagamento

Novos gateways sempre via adapter pattern (`src/gateway-pagamento/`).
Nunca chamar AsaasService direto de fora do módulo asaas — usar
GatewayPagamentoService. Ver `docs/arquitetura/gateways.md`.

Exceção: `pix-excedente.service.ts` usa AsaasService direto (transferência PIX específica).

## Dados de teste

Todos os dados atualmente no banco dev são de teste. Quando encontrar
dados suspeitos (CPF/CNPJ fake padrão 1-2-3-4, nomes 'REMOVIDO', emails
@removido.invalid, domínios @test ou @invalid), pode deletar sem perguntar.
Apenas mostra a lista do que vai deletar e executa. Isso vale até o primeiro
parceiro real entrar em produção.

## Regra de atualização

Se estado mudar muito (muitos tickets fechados, novo sprint, schema grande),
re-gerar RAIO-X e atualizar `docs/COOPEREBR-ALINHAMENTO.md`. Avisar Luciano antes.

## Regras de segurança para migrations e alterações de schema

Qualquer alteração de schema que envolva os casos abaixo EXIGE auditoria
**prévia** dos dados afetados:

1. Mudança de tipo de campo (String → Enum, String → Int, etc)
2. Tornar campo obrigatório (NULL → NOT NULL)
3. Deletar campo existente
4. Alterar default value
5. Renomear campo com impacto em queries
6. Alterar unique/index constraints

### Checklist ANTES de aplicar qualquer dos casos acima

**A.** Rodar SELECT que conta:
- Quantos registros têm valor não-nulo no campo
- Distribuição de valores (`SELECT valor, COUNT(*) GROUP BY`)
- Valores que não vão sobreviver à mudança

**B.** Reportar ao Luciano o que será perdido (se algo) e pedir
autorização explícita antes de executar.

**C.** Preferir migração em 2 passos quando possível:
- Passo 1: UPDATE pra normalizar valores existentes
- Passo 2: ALTER TABLE (tipo, NOT NULL, etc)

**D.** Evitar `prisma db push` cego em casos acima — preferir `migrate dev`
com review do SQL gerado. **Nunca** usar `--accept-data-loss` sem
auditoria prévia explícita.

**E.** Em scripts de normalização de dados: sempre dry-run primeiro,
mostrar ANTES/DEPOIS de cada registro, aguardar aprovação.

**F.** Se Luciano pedir "investigar relacionamentos antes de alterar",
auditar TODOS os campos afetados, não só o campo principal da solicitação.

Regra criada após incidente de 2026-04-26 (Sprint 11 Bloco 1): 96 valores
textuais de `Uc.distribuidora` foram perdidos em migration String → Enum
sem auditoria prévia. Registrado no MAPA-INTEGRIDADE-SISTEMA.md.

## Infraestrutura local — backend gerenciado por PM2

O backend roda sob **PM2** como `cooperebr-backend` (id 0). Não é processo
livre via `npm run start:dev`.

**Comandos corretos:**

| Ação | Comando |
|---|---|
| Ver status | `pm2 list` |
| Parar | `pm2 stop cooperebr-backend` |
| Subir (se stopped) | `pm2 start cooperebr-backend` |
| Reiniciar | `pm2 restart cooperebr-backend` |
| Ver logs | `pm2 logs cooperebr-backend --lines 30` |

**NUNCA usar `npm run start:dev` direto.** Mesmo que o usuário diga
"matei o backend", o PM2 pode ressuscitar o processo automaticamente,
criando processos zumbi e bloqueio do `query_engine_bg.wasm` (ou
`.dll.node` em versões antigas) do Prisma.

### Regras pra `prisma generate` / `db push`

**OBRIGATÓRIO** antes de `prisma generate` ou `prisma db push`:

1. `pm2 stop cooperebr-backend`
2. Confirmar porta 3000 livre: `netstat -ano | findstr :3000` (não deve
   ter `LISTENING`)
3. Rodar `prisma generate` / `db push`
4. `pm2 restart cooperebr-backend`

**Sem parar o PM2**, o engine Prisma fica lockado e o `EPERM` persiste
mesmo matando processo manualmente — PM2 respawna instantaneamente.

### REBUILD obrigatório quando muda código backend

PM2 roda `dist/src/main.js` (build compilado), **NÃO ts-node em modo watch**.
Mudanças em arquivos `.ts` **não chegam ao runtime** sem rebuild.

Sequência correta após qualquer mudança em `backend/src/`:
1. `pm2 stop cooperebr-backend` (libera locks)
2. `cd backend ; npm run build` (regenera `dist/`)
3. `pm2 restart cooperebr-backend`

Sintomas de "esqueci de rebuildar":
- 404 em endpoints novos
- Erros Prisma referenciando campos já deletados (`P2022 column 'X' does not exist`)
- Validação `tsc --noEmit` passa mas runtime falha

`scripts/` está excluído do build (`tsconfig.build.json`) — utilitários standalone
que rodam via `ts-node` direto, não vão pro `dist/`.

**Prisma v6** usa `query_engine_bg.wasm` (não mais `.dll.node`). Engine
binário antigo (`query_engine-windows.dll.node` de versões anteriores) é
**lixo no disco** e pode ser ignorado — verifique a data do `.wasm` pra
saber se o regenerate funcionou, não a do `.dll`.

Regra criada após sessão de 2026-04-25, onde 1h foi gasta debugando
erros 500 em `/ocorrencias` e `/contratos` que eram só engine Prisma
antigo carregado em memória pelo backend que o PM2 mantinha respawnado.

### Frontend Next.js — `next start` (build de produção) SOB PM2 (corrigido 2026-06-04)

⚠️ **CORREÇÃO 2026-06-04:** o frontend NÃO roda em `npm run dev`/HMR. Roda em
**`next start -p 3001`** (build de produção do `.next/`) gerenciado pelo **PM2 como
`cooperebr-frontend`**. **NÃO HÁ HMR.**

**Consequência crítica:** **toda** mudança em `web/` (qualquer `.tsx`, não só páginas
novas) só aparece no browser **após rebuild + restart**:
```
cd web ; npm run build ; pm2 restart cooperebr-frontend
```
Depois, **hard reload** no browser (Ctrl+Shift+R).

Sintoma de "esqueci de rebuildar o frontend": a edição está no arquivo-fonte (confirma com
grep) mas o browser mostra a versão antiga — o bundle servido foi compilado antes da mudança.

Recuperação se cair: `pm2 restart cooperebr-frontend` (ou `pm2 start` se stopped).
Origem: sessão 2026-06-04 — Fatia A (nomenclatura CooperToken no portal) não aparecia no
browser porque o frontend estava em `next start`, não `next dev`. ~30min até diagnosticar.

**Cuidado com VS Code reload:** ao reabrir VS Code/terminal integrado
e usar histórico do PowerShell, é fácil rodar comando velho de PM2
stop/start sem perceber que o backend já está estável. Resultado: sobe
restart count desnecessário. Antes de rodar `pm2 stop/start/restart`,
sempre `pm2 list` pra ver o estado real primeiro.

Regra criada após sessão de 2026-04-28: PM2 do `cooperebr-backend`
chegou a 331 restarts acumulados (alguns pela manhã por node órfão,
outros à noite por reaproveitamento acidental do histórico PowerShell).

## Estado atual do projeto (atualizado 2026-04-28)

Sprint 13a P0 + Dia 1 concluídos. Painel SISGD operacional em `/dashboard/super-admin`.

**Banco final:**
- 2 cooperativas: **CoopereBR** (produção, plano OURO, 307 cooperados / 299 ATIVOS) + **CoopereBR Teste** (TRIAL, plano PRATA, 4 cooperados ATIVOS)
- 1 FaturaSaas PENDENTE (CoopereBR Teste, R$ 5.900, vencida 10/04 — para validar painel de inadimplência)
- AuditLog table criada (vazia — ativação no Sprint 13b com interceptor)
- 4 índices cross-tenant criados em `cobrancas`, `cooperados`, `faturas_saas`

**Sprint 13a Dia 1 entregou:**
- `MetricasSaasService` + endpoint `GET /saas/dashboard` (gated SUPER_ADMIN)
- Tela `/dashboard/super-admin` com 5 cards (parceiros, membros, faturado, MRR, alerta inadimplência + hero incêndios)
- Sidebar reorganizada com link "Painel SISGD" em "Gestão Global"
- Refactor `gerarFaturaParaCooperativa` exposto como público (commit `0d53773`)

**Conquistas históricas do Sprint 10 (preservar):**
- Primeiro email SMTP funcional (email_logs.status=ENVIADO passou de 0 pra 1+)
- Primeiro WhatsApp automático pós-reativação
- LGPD compliance (whitelist dev + flag ambienteTeste + 112 registros mascarados)
- CADASTRO_V2 desbloqueado

**Conquistas Sprint 11 e 12:**
- Sprint 11: Arquitetura UC consolidada (numero/numeroUC/distribuidora/numeroConcessionariaOriginal), pipeline OCR multi-campo, E2E fatura Luciano
- Sprint 12: Webhook Asaas validado em sandbox + 3 bugs corrigidos (CLUBE dupla bonificação, percentualDesconto, dataVencimento)

Documentos vivos permanentes (ler ao iniciar sessão):
- docs/MAPA-INTEGRIDADE-SISTEMA.md (atualizar a cada sprint)
- docs/PLANO-ATE-PRODUCAO.md (roteiro de sprints até produção)
- docs/COOPEREBR-ALINHAMENTO.md
- docs/PRODUTO.md (visão humana do produto — substitui SISGD-VISAO movido pra histórico em 03/05/2026)
- docs/debitos-tecnicos.md (P1/P2/P3 vivos)
- docs/especificacao-clube-cooper-token.md
- docs/especificacao-contabilidade-clube.md
- docs/especificacao-modelos-cobranca.md
- CLAUDE.md (este arquivo)

Próximo passo: **Sprint 13a Dia 2** — lista de parceiros enriquecida com filtros e smoke test. Frase de retomada: "Iniciando Sprint 13a Dia 2 — lista parceiros + filtros".

---
> Source: [lucbragatto/cooperebr](https://github.com/lucbragatto/cooperebr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
