## f3f-auto-ads

> Ferramenta SaaS para criação e publicação automatizada de anúncios no Meta Ads (Facebook/Instagram). Integra com a Meta Graph API **v25.0** via Supabase Edge Functions. Usuários são gestores de tráfego que sobem campanhas rapidamente sem abrir o Gerenciador de Anúncios. Quatro presets de campanha:

# F3F AUTO-ADS — Guia para Claude

## O que é este projeto

Ferramenta SaaS para criação e publicação automatizada de anúncios no Meta Ads (Facebook/Instagram). Integra com a Meta Graph API **v25.0** via Supabase Edge Functions. Usuários são gestores de tráfego que sobem campanhas rapidamente sem abrir o Gerenciador de Anúncios. Quatro presets de campanha:

- **FASE 1** — Tráfego para perfil do Instagram (`OUTCOME_TRAFFIC`, `PROFILE_VISIT`, `INSTAGRAM_PROFILE`)
- **FASE 2** — Engajamento de vídeo p/ montar públicos (`THRUPLAY`, `ON_VIDEO`) — alimenta audiências VV
- **FASE 3** — Leads via WhatsApp (`OUTCOME_LEADS`, `CONVERSATIONS`, `WHATSAPP`); variante **Vendas** usa `OUTCOME_SALES`
- **L.T** — Tráfego p/ site / conversões (`OFFSITE_CONVERSIONS`, `WEBSITE`) com nomenclatura própria

## Stack

| Camada | Tecnologia |
|--------|-----------|
| Frontend | React 18.3 + TypeScript 5.8 + Vite 5.4 (plugin SWC) |
| Roteamento | React Router 6.30 (páginas lazy) |
| Estado servidor | TanStack Query 5.83 |
| Formulários | React Hook Form 7.61 + Zod 3.25 |
| UI | shadcn/ui (Radix) + Tailwind 3.4 (`tailwindcss-animate`) |
| Auth | Supabase Auth (email/senha) — `@supabase/supabase-js` 2.98 |
| Backend | Supabase Edge Functions (Deno/TypeScript) |
| Testes | Vitest 3.2 + Testing Library + jsdom |
| API Meta | Meta Graph API **v25.0** (nunca usar outra versão) |

## Estrutura de diretórios

```
src/
  pages/          # Index, LoginPage, AdminPage, ResetPasswordPage, SettingsPage, MetaCallback, NotFound
  components/     # PublishForm (core, ~2450 linhas; contém child LogPanel)
                  # Header, Fase3Components, LocationSelector, SearchableSelect,
                  # IDDisplay, NavLink, ProtectedRoute, + ui/ (48 componentes shadcn)
  contexts/       # AuthContext (Supabase), ThemeContext (dark/light)
  hooks/          # use-toast, use-mobile
  lib/            # meta-api.ts (client das edge functions), naming.ts (geradores de nome),
                  # admin.ts (isCurrentUserAdmin / inviteUser), utils.ts (cn)
  integrations/   # supabase/client.ts + supabase/types.ts (gerados)

supabase/functions/   # 21 functions + _shared/
  # ── Publicação ──
  meta-publish/             # Edge principal — orquestra campaign > adset > creative > ad (1 chamada por criativo)
  meta-publish-validate/    # Pré-voo: cria adset de teste, valida targeting/budget, limpa
  meta-validate-creative/   # Valida link IG (media_id) ou arquivo do Drive (Range via API key)
  # ── OAuth / Token ──
  meta-login/               # Redirect OAuth Meta (scopes incl. whatsapp_business_management)
  meta-oauth-callback/      # Troca code → token curto → token longo; grava em meta_connections
  meta-status/              # Checa validade/expiração do token; auto-refresh se faltam ≤7 dias
  meta-token-refresh-cron/  # Cron diário: renova tokens que expiram em ≤14 dias (CRON_SECRET)
  # ── Descoberta (popula dropdowns) ──
  meta-ad-accounts/         # Lista contas + descobre IG accounts via páginas
  meta-audiences/           # Públicos custom + saved (paginado)
  meta-campaigns/           # Campanhas ACTIVE (detecta CBO vs ABO)
  meta-whatsapp-numbers/    # Descobre números WABA (promote_pages como fonte primária)
  meta-message-templates/   # Templates WhatsApp extraídos de creatives CTWA existentes
  meta-pixels/              # Lista pixels da conta
  meta-location-search/     # Busca geográfica
  # ── FASE 2 ──
  meta-create-video-audience/  # Cria custom audience VV50% de um vídeo
  # ── Diagnóstico (debug, não-produção) ──
  meta-ad-review/           # Lê ad_review_feedback + effective_status p/ diagnosticar rejeição
  meta-adset-diff/          # Compara configs de adsets
  meta-campaign-diagnostic/ # Dump de estrutura campaign/adset/ad/creative
  meta-fase1-diagnostic/
  meta-fase3-diagnostic/    # Testa variações de promoted_object WhatsApp (PAUSED, limpa)
  # ── Admin ──
  admin-invite-user/        # Cria gestor no Auth + dispara email de convite (requer app_admins)
  _shared/email.ts          # Helper Resend (sendInviteEmail)
```

## Fluxo de publicação (`meta-publish`)

```
1. Validar identidade (page_id + instagram_actor_id)
2. Resolver mídia por criativo → buildOne() mapeado em paralelo (1 chamada cada)
3. Criar Campaign (ACTIVE) — checkpoint manual removido (2026-07-06); entrega imediata ao publicar
4. Criar Adset (ACTIVE) — com promoted_object correto por preset
   (FASE 2 Completo: 1 criativo + N adsets, um por audiência; FASE 2 Adaptado: N criativos + 1 adset por criativo, cada um combinando 2-10 públicos)
5. Criar Adcreative
6. Criar Ad (ACTIVE)
```

### Resolução de mídia

- **IG shortcode/reel** → `source_instagram_media_id` (query direta na media do IG account)
- **Drive vídeo** → **primário**: `GOOGLE_DRIVE_API_KEY` + `file_url` no FormData → **Meta baixa o arquivo direto** (evita OOM 546 na edge). Faz poll do status do vídeo **5x** no fast-path `file_url` (index.ts:687) / **4x** no fallback multipart (index.ts:807).
  **Fallback** (sem API key ou `file_url` falha): edge bufferiza os bytes e sobe via multipart `source`.
  O arquivo do Drive precisa estar **público** ("qualquer pessoa com o link"); senão `meta-validate-creative` barra antes de publicar.

## Regras críticas por preset

### FASE 1 (tráfego p/ perfil IG)
- Adset: `optimization_goal: PROFILE_VISIT` + `destination_type: INSTAGRAM_PROFILE` — **NÃO** `VISIT_INSTAGRAM_PROFILE` (revertido 2026-07-04: diagnóstico contra campanha gabarito real mostrou que a Meta só anexa o `tracking_specs` `action.type=visit_instagram_profile` no ad — sinal de reconhecimento do goal — sob `PROFILE_VISIT`; `VISIT_INSTAGRAM_PROFILE` faz o sistema entregar pior sem erro nenhum, silenciosamente)
- `promoted_object`: `{ page_id, instagram_actor_id }` — o `instagram_actor_id` (= `igActorId` resolvido por conta) foi adicionado em 2026-07-27: diff de 3 adsets reais mostrou que o gabarito manual (ACTIVE) tinha esse campo e os do sistema não → causava #2016153 "not eligible for Profile Visit". **NÃO** enviar `instagram_profile_id` (campo diferente; causa #1346001). Ver comentário em `buildFase1Adset`.
- `attribution_spec: [{ event_type: CLICK_THROUGH, window_days: 1 }]` (presente no gabarito; sem isso a Meta usa janela default)
- `targeting_automation: { advantage_audience: 1 }` — **fixo, sempre, incondicional** (`buildFase1Targeting`, index.ts:~2092). Decisão do usuário 2026-07-28 REVOGOU o espelhamento do público salvo (#77, 2026-07-10): o mirror saía `adv=0` nos publishes reais (público salvo com adv=0) e não batia com o gabarito manual, que roda `adv=1` e entrega melhor. Com `adv=1` a idade muda de canal: `age_range` é a SUGESTÃO (fonte = público salvo quando único/`saved`, senão o próprio `targeting`) e `age_min`/`age_max` viram o CONTROLE rígido, normalizado p/ **18/65** (evita 100/1870188). Ver comentário index.ts:2062-2082.
- Creative: `source_instagram_media_id` + `instagram_user_id` + `call_to_action: VIEW_INSTAGRAM_PROFILE`

### FASE 2 (engajamento de vídeo)
- `optimization_goal: THRUPLAY` + `destination_type: ON_VIDEO`
- Dois modos (dispatch por `body.fase2_combined_adset`, index.ts:3040):
  - **COMPLETO** (`fase2_combined_adset` ausente/false): **1 criativo + N adsets** — um adset por audiência; cada adset exige `audience_id` de inclusão (`buildFase2Adset`, `custom_audiences` com 1 id).
  - **ADAPTADO** (`fase2_combined_adset === true`): **N criativos + 1 adset por CRIATIVO** — cada adset combina **2-10 públicos** no MESMO conjunto (`buildFase2AdsetCombined`, index.ts:2506; `custom_audiences` com N ids = a Meta combina como OR).
- Campos do adset FASE 2 (ambos os modos): `attribution_spec: [{ CLICK_THROUGH, 1 }]` (index.ts:2482), `targeting_automation: { advantage_audience: 0 }` (2467), `targeting_relaxation_types: { lookalike: 0, custom_audience: 0 }` (2468), `geo_locations: { countries: ["BR"], location_types: ["home","recent"] }` (2464).
- Usa audiências VV50% criadas por `meta-create-video-audience`
- Creative: vídeo do Drive (re-upload) ou post/reel do IG

### FASE 3 (leads via WhatsApp)
- `optimization_goal: CONVERSATIONS` + `destination_type: WHATSAPP`
- `promoted_object`: obrigatórios `{ page_id, whatsapp_phone_number }`; o código monta `{ page_id, smart_pse_enabled: false, whatsapp_phone_number }` (index.ts:2202-2206). **NÃO** enviar `whats_app_business_phone_number_id` — a Meta rejeita com **2446886**. `validateFase3PromotedObject` tem allow-list de 5 chaves (index.ts:404-406): `page_id`, `whatsapp_phone_number`, `smart_pse_enabled`, `pixel_id`, `custom_event_type` — qualquer chave fora dela HARD-BLOCKA o publish. Variante Vendas ZAP soma `pixel_id` + `custom_event_type` (2208-2211)
- `attribution_spec: [{ event_type: CLICK_THROUGH, window_days: 1 }]`
- `targeting_automation: { advantage_audience: 0, individual_setting: { age: 0, gender: 0 } }` (desativado — confirmado com usuário 2026-07-02; público rígido, sem sugestão da Meta; index.ts:2295-2298)
- Creative: `source_instagram_media_id` + `instagram_user_id` + `call_to_action: WHATSAPP_MESSAGE` + `page_welcome_message` (JSON de autofill da conversa WhatsApp; index.ts:1516)
- Ad: inclui `tracking_specs` para onsite_conversion, messenger e whatsapp
- **Variante Vendas**: `objective: OUTCOME_SALES` + pixel/`PURCHASE` no `promoted_object`

### L.T (tráfego/conversão p/ site)
- `optimization_goal: OFFSITE_CONVERSIONS` + `destination_type: WEBSITE`
- Nome de campanha próprio (`generateLtCampaignName` em `naming.ts`); suporta ABO/CBO
- Atribuição depende do objetivo do preset:
  - **L.T Vendas** (`fase3-vendas-lp`, `OUTCOME_SALES`) → `is_incremental_attribution_enabled: true` e **NÃO** manda `attribution_spec` (são mutuamente exclusivos; index.ts:2412, 2427-2431).
  - **WEBSITE/LEAD** (`fase3-leads-lp`, `OUTCOME_LEADS`) → `attribution_spec: [{ CLICK_THROUGH, 7 }]` (index.ts:2407).
  - O modelo rico `[CT7, VT1, ENGAGED_VIDEO_VIEW 1]` só existe no branch `custom_event_type === "PURCHASE"` (index.ts:2401-2406), que o caminho `OUTCOME_SALES` (incremental) não usa.
- Advantage+ esconde a seleção de público (Meta acha sozinho)

## DSA / Anunciante (campanhas BR)

- Fonte **PRIMÁRIA** do beneficiário/pagador: `regional_regulation_identities { universal_beneficiary, universal_payer }` (ID da entidade VERIFICADA) lido de um adset já existente da conta e replicado (index.ts:1807, 1813-1817, 1872-1875). `/dsa_recommendations` é só **FALLBACK**, buscado apenas quando não há entidade verificada herdável nem beneficiário informado pelo usuário (index.ts:1829-1846). Nunca usar `page_id` numérico.
- `applyDsa(p)` roda **incondicionalmente** em TODOS os builders de adset (index.ts:2164/2322/2432/2485/2536) — **não** existe gate por Advantage+/BR no código. (O dump de gabarito mostrava 192 adsets Advantage+ / 0 com DSA, o que sugeria pular o beneficiário em Advantage+ BR, mas o código não implementa esse gate.)
- Caminho primário envia só anunciante **verificado** (por ID). O **FALLBACK**, porém, envia `dsa_beneficiary`/`dsa_payor` em **texto livre não-verificado** (index.ts:1876-1879).

## Erros Meta conhecidos

| Code | Subcode | Causa | Fix |
|------|---------|-------|-----|
| 100 | 2446391 | Ad rejeitado — creative incompatível com adset | Causa não é `instagram_profile_id` ausente (gabarito real funciona sem ele) — investigar CTA/goal/destination_type coerentes antes de mexer no `promoted_object` |
| 100 | 3858634 | "Anunciante ausente" (DSA, BR) — é `error_subcode` do code **100** (index.ts:2952), não um code de topo | Verificação de anunciante no Business Manager — **não resolve por API** |
| 1,2,4,17,32,341,613 | — | `TRANSIENT_META_CODES` (index.ts:45): `isTransientMeta()` marca TODOS como transiente/`rate_limited` (o **frontend** usa isso p/ decidir retry) | Mas o loop de criação de adset na edge (index.ts:2877) só REtenta **[2, 4, 17]** com backoff `[2s, 6s, 15s]` (index.ts:2591); os demais (1, 32, 341, 613) só marcam `rate_limited`/mensagem e **NÃO** retentam (`break`) |

## Variáveis de ambiente

```
# Frontend (Vite)
VITE_SUPABASE_URL
VITE_SUPABASE_PUBLISHABLE_KEY
VITE_SUPABASE_PROJECT_ID        # presente no .env; sem referência no código

# Edge functions (Supabase secrets)
SUPABASE_URL
SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY
META_APP_SECRET                 # troca de token OAuth
GOOGLE_DRIVE_API_KEY            # Drive vídeo via file_url (Meta baixa direto)
RESEND_API_KEY                  # email de convite de gestores
RESEND_FROM_EMAIL               # ex: "F3F AUTO-ADS <noreply@agenciaf3f.com.br>"
APP_URL                         # link no email; fallback: https://console.agenciaf3f.com.br
CRON_SECRET                     # autentica meta-token-refresh-cron
```

> Meta **App ID `910343951738258`** está hardcoded em `meta-login` e `meta-oauth-callback`.

## Comandos

```bash
npm run dev        # Dev server porta 8080 (host ::)
npm run build      # Build produção (Vite)
npm run lint       # ESLint
npm run test       # Vitest (run único)
npm run test:watch # Vitest watch
```

> **Antes de commitar frontend**: rodar `npx tsc -p tsconfig.app.json`. ⚠️ `npx tsc --noEmit` (sem `-p`) é **no-op** — o `tsconfig.json` raiz tem `files: []`, então não checa nada. O Vite build também NÃO roda `tsc` — refs órfãs/erros de tipo passam no build e crasham em runtime.

## Banco de dados (Supabase)

Projeto de produção: ref **`csfpqioxmsocdqavwkvn`**. 4 migrations em `supabase/migrations/`. Tabelas:

- `profiles` — display name por usuário. RLS por `auth.uid()`.
- `meta_connections` — token OAuth do gestor. **UNIQUE(user_id)** = 1 conexão por gestor. RLS por `auth.uid()`. Índice em `expires_at`.
- `publish_jobs` — deduplicação/idempotência (fingerprint SHA-256) + status. Índices `(user_id, created_at)` e `(user_id, status)`.
- `message_templates` — templates WhatsApp (greeting, ready_message). RLS por `auth.uid()`.
- `app_admins` — usuários que podem convidar gestores. RLS apenas SELECT da própria linha.

> A conexão Meta é **compartilhada** (token do admin `agenciaf3f@gmail.com`); o app puxa WhatsApp/contas por essa conexão. Precisa do scope `whatsapp_business_management` nela.

## Deploy

### Edge functions
```bash
supabase functions deploy meta-publish     # (prod ref: csfpqioxmsocdqavwkvn)
```
`supabase/config.toml` define `verify_jwt = false` nas functions públicas.

### Frontend (Vercel)
- Projeto **`f3f-auto-ads`**, team scope **`agenciaf3f-2309s-projects`** (orgId `team_zVUz7ywTl2lCsc6ULiPbRzD3`, ver `.vercel/project.json`).
- Deploy **manual via CLI** (git integration off — push não auto-deploya):
  ```bash
  vercel --prod --yes --scope agenciaf3f-2309s-projects
  ```
- Produção em **console.agenciaf3f.com.br** (migrou de `ads.` em jun/2026 p/ fugir de adblock; `ads.` ainda é alias). DNS no Cloudflare, proxy **off**.
- `vercel.json` só tem rewrite SPA (`/(.*) → /index.html`).
- Se um fix "não apareceu em produção": checar se `.vercel/project.json` aponta pra team certa (já esteve stale).

## Multi-tenancy (gestores isolados)

Cada gestor tem conta própria no Supabase Auth. Sessões, conexões e publicações isoladas por `auth.uid()` via RLS + filtros nas edge functions. **Signup público desabilitado** — onboarding por convite de admin.

### Onboarding de gestor (admin)
1. Login como admin (qualquer user em `public.app_admins`).
2. Header → "Admin" → Nome + Email → "Enviar convite".
3. `admin-invite-user` cria o user no Auth com senha provisória e dispara email via Resend.
4. Gestor faz login; opcionalmente troca senha em "Esqueci minha senha" (`ResetPasswordPage`).

### Como adicionar um novo admin
```sql
INSERT INTO public.app_admins (user_id)
SELECT id FROM auth.users WHERE email = 'novo@admin.com';
```

### Configuração inicial pós-deploy
```bash
supabase db push

supabase secrets set RESEND_API_KEY=re_xxxxxxxxxxxx
supabase secrets set RESEND_FROM_EMAIL="F3F AUTO-ADS <noreply@agenciaf3f.com.br>"
supabase secrets set APP_URL="https://console.agenciaf3f.com.br"
supabase secrets set META_APP_SECRET=xxxxxxxx
supabase secrets set GOOGLE_DRIVE_API_KEY=xxxxxxxx
supabase secrets set CRON_SECRET=xxxxxxxx

supabase functions deploy admin-invite-user

# Painel Supabase → Authentication → Settings:
#   - Desabilitar "Enable new user signups"
#   - Site URL: https://console.agenciaf3f.com.br

# Designar admin inicial:
INSERT INTO public.app_admins (user_id)
SELECT id FROM auth.users WHERE email = 'agenciaf3f@gmail.com';
```

## Tooling Claude neste projeto (skills ativas)

Recursos disponíveis em sessões Claude Code neste repo:

- **MCP Supabase** (`mcp__claude_ai_Supabase__*`): `deploy_edge_function`, `apply_migration`, `execute_sql`, `get_logs`, `get_advisors`, `list_tables`, `list_migrations` — usar p/ deploy/inspeção sem CLI. Prod ref `csfpqioxmsocdqavwkvn`.
- **MCP Meta-ADS** (`mcp__claude_ai_Meta-ADS__*`): apenas **gabarito/oráculo na sessão** — NÃO roda no app em runtime. O app usa a Graph API direta nas edge functions. Não tentar plugar no runtime.
- **MCP Google Drive** (`mcp__claude_ai_Google_Drive__*`): inspecionar permissão/metadados dos vídeos de criativo (público vs privado).
- **Skills**:
  - `context-mode` — processar outputs grandes (logs, builds, diffs) sem estourar contexto.
  - `caveman` — modo de comunicação conciso (ativo nesta sessão).
  - `vercel:*` — deploy, CLI, env (`vercel:deploy`, `vercel:env`, `vercel:vercel-cli`).
  - `code-review` / `ultrareview` — revisão de diff/branch (multi-agente na nuvem).
  - `karpathy-guidelines` — diretrizes de mudança cirúrgica/simplicidade.
- **Memórias** relevantes (em `~/.claude/projects/.../memory/`): `meta_graph_api_version` (sempre v25.0), `mcp_meta_as_oracle`, `whatsapp_resolution`, `drive_video_publish`, `dsa_advertiser_verification`, `saved_audience_api_limitation`, `vite_no_tsc_runtime_crash`, `deploy_target`.

## Contexto comercial

Produto voltado para venda em massa. Gestores de tráfego sobem campanhas FASE 1/2/3 e L.T rapidamente sem acessar o Gerenciador de Anúncios do Meta.

---
> Source: [agenciaf3f-droid/f3f-auto-ads](https://github.com/agenciaf3f-droid/f3f-auto-ads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
