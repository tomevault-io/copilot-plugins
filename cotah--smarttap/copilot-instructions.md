## smarttap

> Sistema all-in-one de reputação, fidelidade e relacionamento direto com clientes para pequenos negócios locais na Irlanda. Combina hardware físico impresso em 3D com plataforma SaaS multi-tenant.

# SmartTap — Contexto Completo do Projeto

## O QUE É O SMARTTAP

Sistema all-in-one de reputação, fidelidade e relacionamento direto com clientes para pequenos negócios locais na Irlanda. Combina hardware físico impresso em 3D com plataforma SaaS multi-tenant.

**Tagline:** "Você já pagou para trazer esse cliente uma vez. O SmartTap faz ele voltar."
**Tagline produto:** "TAP. CONNECT. GROW."
**Domínio:** smarttap.ie
**Fundador:** Henrique — indie developer baseado em Dublin, Irlanda

---

## OS 3 PILARES

- **Tap & Review:** NFC/QR → Google Review em 1 clique, sem app
- **Tap & Return:** Stamps digitais sem app → recompensas por fidelidade
- **Tap & Connect:** Coleta contato GDPR-compliant → campanhas diretas

---

## HARDWARE FÍSICO

- Impressora 3D Creality Hi CFS multi-color em Dublin
- 400 NFC tags disponíveis imediatamente
- Custo de produção: ~€0,55 por unidade | Preço: €79-€249 setup
- **3 formatos:**
  - Counter Stand (balcão) — 80x80x60mm
  - Table Tent (mesa) — 90x60mm triangular
  - Wall Plaque (parede) — 120x80mm
- **10 cores PLA:** Preto, Branco, Cinza, Navy, Roxo, Vermelho, Amarelo, Stone Age, Real Green, Forest Green

---

## IDENTIDADE VISUAL (NÃO ALTERE SEM APROVAÇÃO)

> ⚠️ **Rebrand em curso (aprovado 2026-05-31): "SmartTap Dark Electric".** A identidade antiga (verde/âmbar/cream + DM Serif) foi substituída pela paleta dark + ciano elétrico, feel Linear/Vercel. Rollout sequencial: **landing → dashboard → NFC → emails → stand físico**. A landing é a primeira superfície (ver `docs/superpowers/specs/2026-05-31-landing-dark-electric-redesign-design.md`). Superfícies ainda não migradas continuam na paleta antiga até serem feitas.

**Identidade ATUAL (Dark Electric):**
- **Logo:** símbolo ST com ondas NFC — **ciano #00D4FF / branco** — fundo transparente
- **Cores:**
  - Fundo base `#0A0A0F` · Surface `#121219` · Surface-2 `#1A1A24` · Border `#1A2A3A`
  - Accent ciano `#00D4FF` · Ciano profundo `#00BFEA` (hover/gradientes)
  - Texto `#FFFFFF` · Texto secundário `#8899AA` · Superfície clara `#F0FAFE`
  - CTA: ciano `#00D4FF` com texto preto
  - ⚠️ Ciano em fundo claro falha contraste — em superfícies claras só em fills/elementos grandes, nunca texto pequeno.
- **Tipografia:** Geist Sans (headings) + Inter (body) + Geist Mono (code/eyebrow)

**Identidade ANTIGA (arquivada — só em superfícies ainda não migradas):**
- Logo dourado/preto · Verde #1B4D3E · Âmbar #E8A020 · Off-white #F7F5F0 · Preto #1A1A1A · DM Serif Display + DM Sans + JetBrains Mono

---

## STACK TÉCNICO (FIXO — NÃO TROCAR SEM JUSTIFICATIVA FORTE)

```
Arquitetura: Monorepo (Turborepo)

apps/web/      → Next.js 15 + Tailwind + shadcn/ui (PWA — lança agora)
apps/mobile/   → React Native + Expo (estrutura pronta, lança fase 2)
packages/core/ → lógica de negócio compartilhada (TypeScript)
packages/ui/   → componentes compartilhados
packages/api/  → cliente HTTP compartilhado

backend/       → Python + FastAPI

Database:    Supabase (PostgreSQL + RLS multi-tenant)
Auth:        Supabase Auth
Hosting:     Railway (backend) + Vercel (frontend)
Payments:    Stripe
Emails:      Resend
WhatsApp:    Meta WhatsApp Business Cloud API (Graph API direto)
SMS:         Twilio (apenas SMS — ex. OTP de cliente na Sprint 5.6)
Analytics:   PostHog
Errors:      Sentry
```

**Decisão arquitetural importante:** Monorepo com packages compartilhados garante migração web→mobile sem refactor. 80% do código será reaproveitado na fase 2.

---

## PRICING (v2 — aprovado Fase 3)

| Plano | Setup | Mensal | Anual (2 meses grátis) | Clientes |
|---|---|---|---|---|
| SmartReview | €49 | €29/mês | €290 | até 200 |
| SmartLoyalty | €79 | €59/mês | €590 | até 500 |
| SmartPro | €149 | €99/mês | €990 | ilimitado |
| SmartNetwork | €299 | €179/mês | €1.790 | multi-localização |

**Oferta Founding Member (primeiros 5 clientes):**
- Stand custom GRÁTIS + 60 dias grátis + €29/mês vitalício
- Em troca: depoimento em vídeo + 2 indicações nominais

**Oferta Early Adopter (clientes 6-20):**
- €49 setup + 30 dias grátis + €29/mês por 12 meses garantidos

**Meta de upsell:** 40% dos SmartReview migram para SmartLoyalty em 90 dias (chave da unit economics).

---

## MERCADO E ICP

**Foco primeiros 90 dias:** Barbearias Dublin
- 300+ prospects em Dublin
- Decisão rápida, dono presente, dor clara
- Ticket: €79 setup + €39/mês

**Hierarquia de ICPs (revisada Fase 1):**
1. Barbearias Dublin (primário — 300+ prospects)
2. Cafés especialidade / 3rd wave Dublin (independentes, não redes)
3. Pet Grooming Dublin (perfil similar a barbearia, frequência mensal)
4. Salões pequenos (1-2 cadeiras, dono presente)
5. Tattoo Studios pequenos (dono techy, adoram reviews)

Hostels e Clínicas Dentárias foram movidos para **Fase 3+** (loyalty irrelevante para turistas one-off; clínicas têm decisão lenta + compliance).

**Concorrentes principais:**
- SQUID Loyalty (dados deles, sem white-label)
- Stamp Me (app obrigatório)
- ReviewsCard (one-off, sem SaaS)

**Diferenciação:** único com hardware + reviews + loyalty + white-label + dados próprios + presença local Dublin

---

## DATABASE SCHEMA (MVP)

```sql
tenants (id, name, slug, subdomain, logo_url, primary_color, business_type,
         google_review_url, stamps_for_reward, reward_description,
         plan, stripe_customer_id, stripe_subscription_id, trial_ends_at, is_active)

nfc_tags (id, tenant_id, tag_uuid, format, color, location_name,
          active_campaign_id, is_active)

taps (id, tag_id, tenant_id, customer_id, device_type, interaction_type,
      action_taken, created_at)

customers (id, tenant_id, phone, email, name, birthday,
           gdpr_consent, gdpr_consent_at, total_visits, total_stamps,
           last_visit_at, created_at)

stamps (id, customer_id, tenant_id, tap_id, multiplier, created_at)

rewards (id, customer_id, tenant_id, stamps_used, validation_code,
         redeemed_at, expires_at, created_at)

campaigns (id, tenant_id, name, type, status, config jsonb,
           sent_count, created_at)
```

---

## ROADMAP COMPLETO

### FASE 0 — Validação (Semanas 1-3) 🎯 AGORA
**Objetivo:** Confirmar que pessoas pagam ANTES de construir

**Semana 1 (paralelo):**
- [ ] Imprimir 5 protótipos (3 formatos, 2 cores)
- [ ] Registrar smarttap.ie no Cloudflare + DNS
- [ ] Criar landing page simples (formulário de interesse)
- [ ] Demo digital FUNCIONAL — página /t/[uuid] com stamps animados + botão Google Review real (não pode ser estática)

**Semana 2:**
- [ ] Lista nominal de 32 prospects no Google Maps (5h em 2 sessões — Henrique)
- [ ] Treino do pitch (60s gravado e refinado 3 vezes)

**Semanas 2-3 (campo):**
- [ ] Visitar 32 negócios Dublin (4 visitas/dia × 8 dias) — barbearias prioritárias
- [ ] Captura: Voice Memos pós-visita + planilha Google Sheets
- [ ] **Meta:** 5 positivos + 3 compromissos de piloto
- [ ] **Oferta Founding Member:** stand grátis + 60 dias grátis + €29/mês vitalício

**Critério para avançar para Fase 1:** 3 negócios comprometidos como pilotos founding.

---

### FASE 1 — MVP (Semanas 3-8)
**Objetivo:** 10 clientes pagantes com produto funcional

**Sprint 0 — Setup (Semana 1)**
- [ ] Monorepo Turborepo criado
- [ ] Supabase project (prod + staging) com schema
- [ ] Railway + Vercel configurados
- [ ] Stripe test mode
- [ ] CI/CD GitHub Actions
- [ ] .env.example documentado

**Sprint 1 — NFC Core (Semanas 1-2)**
- [ ] POST /tap/{tag_uuid} — registra tap, retorna redirect
- [ ] Página mobile /t/[uuid] (Next.js, mobile-first)
- [ ] StampCard visual gamificado
- [ ] Botão Google Review com UTM tracking
- [ ] OptIn form GDPR-compliant
- [ ] Magic link para cliente recorrente

**Sprint 2 — Dashboard (Semanas 2-3)**
- [ ] Auth do negócio (email + Google OAuth)
- [ ] Dashboard: membros, visitas, stamps, reviews
- [ ] Lista de clientes com filtros
- [ ] Configuração de recompensa (X stamps = Y prêmio)
- [ ] Validação de recompensa (código 6 dígitos)

**Sprint 3 — Billing (Semanas 3-4)**
- [ ] Stripe Checkout (planos Review, Loyalty, Pro)
- [ ] Webhooks Stripe (ativa/desativa tenant)
- [ ] Trial 30 dias automático
- [ ] Onboarding wizard
- [ ] Emails transacionais (Resend)

**Meta Fase 1 (revisada Fase 3):** 8-15 clientes pagantes ao fim do mês 6 → ~€570 MRR + setup fees acumulados.

---

### FASE 2 — Produto Completo (Meses 2-4)
**Objetivo:** 50 clientes, produto completo, primeiros cases

**Sprint 4 — Campanhas (Semanas 4-6)** ✅ COMPLETO
- [x] Double Stamp (janela de tempo)
- [x] Reactivation (inativos há N dias)
- [x] Relatório mensal PDF automático
- [x] Segmentação de clientes

**Sprint 4.5 — NFC tag CRUD UI (S5-W0, micro-sprint pré-Sprint 5)** ✅ COMPLETO
- [x] Backend CRUD (`/v1/tags`) + Literal[4] formatos + Literal[10] cores PLA
- [x] `/dashboard/tags` com swatch grid + Copy URL + soft-delete toggle
- [x] Remove bloqueador "preciso de SQL manual pra cada signup"

**Sprint 5 — WhatsApp AI Assistant (Semanas 6-10)**

3 features integradas, todas baseadas em WhatsApp (Meta Cloud API) + Claude API.

**Stack:** Meta WhatsApp Business Cloud API (Graph API direto, sem Twilio) + Claude API (`claude-sonnet-4-6` em todas as features de AI) + Google Business API (OAuth p/ posting de respostas) — _decisão revista 2026-05-29: Meta direto em vez de Twilio._

- [ ] **Feature 1: Bot para dono consultar dados via WhatsApp**
  - Dono envia "Quantos clientes esta semana?" / "Qual minha melhor tag?" / "Quem não vem há mais de 30 dias?"
  - Bot autentica via número de telefone vinculado ao `tenant_id`
  - Claude lê a pergunta, traduz pra query nas APIs internas (`/v1/dashboard/overview`, `/v1/customers`, `/v1/segments/preview`), responde em linguagem natural
  - Suporta requests de relatório: "Manda o PDF de maio" → bot anexa o PDF via WhatsApp media (Meta Cloud API)
  - Mesma `CRON_TOKEN`-style auth: lookup de tenant por phone hash, rate-limited

- [ ] **Feature 2: Remarketing automático "tapou mas não deu review"**
  - Cron diário (ou hook pós-tap) detecta clientes com `action_taken=stamp_earned` mas sem `action_taken=review_clicked` num período (ex: 24h após o tap)
  - Envia WhatsApp pro cliente (se `gdpr_consent=true` e tem telefone): "Obrigado pela visita à <business>! Se gostou, deixa uma review aqui: <google_review_url>"
  - Cooldown por cliente (ex: 1 send por mês) pra não virar spam
  - Métrica: % de conversão tap → review depois do nudge

- [ ] **Feature 3: Resposta automática de Google Reviews via Claude**
  - Cron busca novas reviews via Google Business API
  - Claude lê a review + contexto do negócio (tenant name, business_type, tom da marca) e gera resposta
  - 2 modos: **draft** (envia pro dono via WhatsApp da Feature 1 pra aprovar antes de postar) ou **autopost** (posta direto se 4-5 estrelas; envia draft pro dono se ≤3)
  - Alerta especial pra reviews 1-2 estrelas (Feature original do Sprint 5 antigo)

**Pré-requisitos infra:**
- Conta Meta Business + número registado na WhatsApp Cloud API (já temos). Env: `WHATSAPP_ACCESS_TOKEN`, `WHATSAPP_PHONE_NUMBER_ID`, `WHATSAPP_APP_SECRET`, `WHATSAPP_VERIFY_TOKEN`, `WHATSAPP_API_VERSION`
- Google Cloud project + OAuth consent screen + Google Business API enabled
- `ANTHROPIC_API_KEY` env var (backend)
- Phone → tenant mapping table (nova migration)

**Roadmap pós-aprovação Google Business Profile API (pendente):**
- [ ] **Feature 3 Fase B** — autopost (4-5★) + push de draft/alertas de review negativa via WhatsApp (reusa o bot da Feature 1).
- [ ] **Ligar a Feature 3 ao bot do WhatsApp** — novas tools no `bot_actions`/`bot_tools` para o dono **ver e redigir respostas a reviews diretamente no WhatsApp** (hoje o bot só encaminha para o dashboard, porque não tem ferramenta para ler reviews individuais). Só entra **depois** de a Google Business Profile API estar aprovada.

**Sprint 5.6 — Identificação por telefone via OTP (Semanas 10-11)**

Fecha um buraco UX da Sprint 1: cliente que limpa o browser ou troca de telemóvel perde o cookie de identificação e re-aparece como cliente novo, perdendo o histórico de stamps. Solução: na página /t/[uuid] aparece um caminho paralelo "Already a member?" que recupera a conta via SMS OTP — sem precisar criar conta nova, sem precisar lembrar do magic link da última visita. Encaixa após a Sprint 5 porque reutiliza a conta Twilio já configurada (SMS é mesmo Account SID + Auth Token, só muda o canal).

**Problema:** Hoje a identificação do cliente vive num cookie. Se o cliente limpa o browser, troca de telemóvel, ou usa modo anónimo, tapa a tag e re-aparece como cliente novo — stamps anteriores ficam órfãos no histórico do tenant. Único caminho actual é magic link da última visita, irrealista pro caso típico de barbearia ("já não tenho aquele email/SMS").

**Solução:** Quando cliente toca na tag e não tem cookie:
- Mostrar bloco discreto `[Already a member? Enter your phone number]` no formulário opt-in (subordinado ao botão Review, consistente com a hierarquia do redesign /t/[uuid])
- Cliente digita o número → backend procura `customers WHERE phone = ?` no tenant
- Se existe → envia código de 4 dígitos via Twilio SMS, valida, set cookie identificador, restaura stamps
- Se não existe → resposta "code sent" igual (anti-enumeration) + flow normal de opt-in já implementado

**Features:**

- [ ] **Input "Already a member?"** — segundo bloco na `/t/[uuid]` opt-in form, abaixo do signup primário, visualmente mais discreto
- [ ] **POST `/v1/customers/identify-request`** — recebe `{phone, tenant_id}`; envia OTP 4 dígitos via Twilio SMS; persiste em `customer_otp_codes(phone, tenant_id, code_hash, expires_at, attempts)` com TTL 10min; rate-limit (1 send/min, 3 sends/hora por phone+tenant); sempre devolve `{ok:true}` mesmo se phone não existe (anti-enumeration)
- [ ] **POST `/v1/customers/identify-verify`** — recebe `{phone, code, tenant_id}`; valida via constant-time compare contra code_hash; em sucesso, set cookie identificador (mesmo formato do magic-link cookie do Sprint 4-W2) + devolve snapshot do customer + reward_state; em falha, incrementa attempts e bloqueia após 5 tentativas erradas
- [ ] **Fusão de conta anónima com conta identificada** — se o tap actual já criou um stamp anónimo (customer_id null) antes da identificação, transferir esse stamp pra conta identificada (anti-perda; idempotente caso o cliente volte a identificar-se)
- [ ] **Rate limiting + anti-abuse** — bloqueio temporário (1h) após 5 códigos errados consecutivos; máximo 3 OTP requests por phone+tenant por hora; CAPTCHA ou similar se detectar padrão de abuse

**Stack:** Twilio SMS (canal SMS — `messages.create({ from: TWILIO_SMS_FROM, ... })`). Nova migration `customer_otp_codes` table. Reaproveita o cookie de identificação existente da Sprint 1 (formato `customer_session_<tenant_id>`).

> ⚠️ **Nota (2026-05-29):** a Sprint 5 migrou de Twilio para **Meta WhatsApp Cloud API**. Como a Meta **não envia SMS**, a 5.6 (OTP por SMS) precisa da **sua própria conta Twilio SMS** — já não há conta Twilio partilhada da Sprint 5 para reutilizar.

**Pré-requisitos infra:**
- Conta Twilio (SMS) própria + Account SID + Auth Token (a Sprint 5 já não usa Twilio)
- Comprar número Twilio SMS Ireland (~€1/mês)
- `TWILIO_SMS_FROM` env var (backend Railway) — formato E.164 `+353...`

**Considerações:**
- OTP por SMS custa ~€0.04/msg em Irlanda — não é gratuito. Rate-limit + cooldown evitam abuso.
- GDPR: o número só é processado para identificação. Texto claro no input: *"We'll text you a 4-digit code. We won't add you to any list."* — sem opt-in implícito para marketing.
- Anti-enumeration: resposta de `identify-request` é sempre `{ok:true}` mesmo se phone não existe — não revelar quem é cliente do tenant.
- A11y: input do OTP precisa de `inputMode="numeric"` + `autocomplete="one-time-code"` para iOS/Android sugerirem o código direto da notificação SMS.
- O bloco "Already a member?" deve ficar abaixo do CTA Review e do signup primário — consistente com a hierarquia mobile-first do redesign de /t/[uuid] de hoje (Review dominante, loyalty secundário).

**Sprint 6 — Review Intelligence Dashboard (Semanas 11-14)**

Análise automatizada de reviews com IA — extrai padrões, segmenta clientes pelo histórico de reviews, gera insights acionáveis de marketing. Combina reviews do Google Business + TripAdvisor num único dashboard inteligente. Encaixa naturalmente após a Sprint 5 porque reutiliza a Google Business API + Claude API já configuradas pra Feature 3.

**Stack:** Google Business API (já configurada na Sprint 5 Feature 3) + TripAdvisor (API oficial se aprovado, scraping com Firecrawl como fallback) + Claude API (`claude-sonnet-4-6` para análise de sentimento e extração de temas, `claude-opus-4-7` para insights estratégicos e geração de copy)

**Fontes de dados:**
- Google Business Reviews (via Google Business API)
- TripAdvisor (via scraping ou API oficial se disponível)

**Features:**

- [ ] **Feature 1: Padrões identificados automaticamente**
  - O que clientes elogiam mais (ex: "sempre pontual", "boa conversa", "corte rápido")
  - O que reclamam mais (ex: "espera longa", "estacionamento", "preço subiu")
  - Palavras e frases recorrentes agrupadas por sentimento e frequência

- [ ] **Feature 2: Segmentação de clientes por reviews**
  - Clientes que voltam sempre (múltiplas reviews ao longo do tempo)
  - O que os regulares valorizam vs o que os novos clientes destacam
  - Perfil do cliente ideal extraído automaticamente das reviews 5-estrelas (idade, tipo de serviço, horário preferido quando inferível)

- [ ] **Feature 3: Insights de marketing acionáveis**
  - "Os teus clientes falam muito sobre X mas tu nunca mencionas no marketing" — gap analysis contra o conteúdo público do negócio (Instagram, site, Google posts)
  - Sugestões de copy para Instagram / Google posts geradas a partir das próprias palavras dos clientes
  - Alertas em tempo real quando review negativa (≤3 estrelas) aparece — push via WhatsApp pela Feature 1 da Sprint 5

- [ ] **Feature 4: Dashboard visual em `/dashboard/reviews-intelligence`**
  - Word cloud dos temas mais mencionados, filtrável por período e fonte
  - Score de sentimento por categoria (atendimento, ambiente, preço, espera, etc)
  - Evolução temporal — gráfico mensal de sentimento médio + volume de reviews

**Pré-requisitos infra:**
- Google Business API + OAuth já configurados na Sprint 5 Feature 3 — reutilizar
- TripAdvisor: aplicar pra Content API oficial; se rejeitado, usar Firecrawl com cooldown + rate-limit por tenant
- `ANTHROPIC_API_KEY` env var (compartilhado com Sprint 5)
- Job batch pra reprocessar histórico de reviews no momento do signup (cron diário pra reviews novas + reprocessamento incremental quando muda o modelo)

**Apps Mobile — Estrutura** (deslocado pra Sprint 5.5 ou Fase 3)
- [ ] React Native + Expo configurado
- [ ] Compartilhando packages/core e packages/ui
- [ ] Funcionalidade básica (tap, stamps, dashboard)

**Meta Fase 2 (revisada Fase 3):** 35-50 clientes ativos ao fim do mês 12 → ~€1.750 MRR (BASE) / €2.950 MRR (otimista).

---

### FASE 3 — Escala (Meses 4-12)
**Objetivo:** 200 clientes, expansão geográfica

**Sprint 7+ — SmartTap Awards Dublin (premiação anual)**

Evento anual de reconhecimento dos melhores negócios que usam SmartTap em Dublin. Combina alavanca de marketing orgânico (press local + redes sociais) com mecânica de fidelização dos clientes existentes (badge + widget no site deles). Funciona como motor de WoM no ICP barbearia/café Dublin antes do produto ter cases públicos.

**Features no produto:**

- [ ] **Ranking público por categoria** — leaderboard renderizado em `/awards/<ano>` ou `/dashboard/awards/leaderboard`, com opt-in do tenant pra aparecer publicamente; reviews / retenção / stamps como métricas
- [ ] **Badge "SmartTap Award Winner 2026"** — visível no dashboard do vencedor + em campanhas/relatórios geradas pelo tenant (vira sinal de proof no email mensal do dono pros clientes)
- [ ] **Widget embeddable** — `<script src="https://smarttap.ie/widgets/award/<tenant_slug>.js">` ou um snippet `<iframe>`, plotando o badge + categoria + ano no site do vencedor com 1 linha de código
- [ ] **Email automático de notificação** — finalistas (top-5 por categoria) e vencedor recebem template Resend assinado; integra com o pipeline transacional do Sprint 3-W7

**Categorias:**
- Most Google Reviews — por business_type: barbershop / café / salon / pet grooming
- Best Customer Retention — % de clientes que voltam ≥3× no ano
- Best Double Stamp Campaign — uplift medido sobre baseline pré-campanha
- Most Loyal Customer Base — média de visitas/cliente
- Best New Business — tenants criados no ano em curso, top performer

**Execução anual:**
- **Janeiro:** anúncio público das categorias + ranking visível no dashboard de cada tenant ("estás em 3º na categoria X — continua a tapar")
- **Dezembro:** cerimónia em Dublin + troféu físico impresso 3D com logo SmartTap (custo marginal — reaproveita a Creality Hi CFS que já imprime stands)
- **PR:** outreach pra Dublin Live, Dublin Gazette, redes sociais do SmartTap + dos vencedores

**Considerações:**
- Ranking público requer opt-in explícito por tenant (default off pra evitar leak inadvertido de métricas competitivas)
- Métricas precisam ser comparáveis entre tenants — normalizar por idade da conta, business_type, plano (não cruzar barbearia com pet grooming na mesma categoria)
- Widget embeddable é vetor de tráfego: cada vencedor que cola no site = backlink + brand exposure pra SmartTap
- Cerimónia inaugural pode ser leve (pub privado Dublin, 15-20 pessoas) — não precisa ser glamour Awards no primeiro ano

**Objetivo:** marketing orgânico, press local, fidelização dos clientes SmartTap existentes. Não é receita direta — é cobertura editorial + brand authority no nicho Dublin.

---

**Sprint 8+ — Integração com sistemas de reservas**

Crítico pro ICP barbearia/salão em Dublin — esses negócios já usam apps de booking. Plugar o SmartTap nesses sistemas tira fricção do staff e dispara o loop fidelidade automaticamente.

- [ ] **Fresha** (principal em Dublin — maior penetração entre barbearias/salões)
- [ ] **Booksy** (forte em salões pequenos)
- [ ] **Square Appointments** (cafés + alguns barber shops)

**Features comuns (uma por integração):**
- [ ] **Stamp automático pós-visita** — webhook do sistema de reserva quando appointment é `completed` → cria tap event + awarda stamp sem precisar do cliente tocar fisicamente o NFC
- [ ] **Review request automático** — N horas após appointment completo, manda WhatsApp/SMS (reutiliza pipeline da Feature 2 do Sprint 5) com link de review
- [ ] **Sincronização de dados de cliente** — pull customer list inicial do sistema de reserva pra migrar cliente existente do barber pro SmartTap sem perder histórico

**Considerações:**
- Cada integração = OAuth/API key flow próprio + webhook receiver + mapper de cliente (phone como chave natural)
- Fresha e Booksy têm API public mas algumas features são partner-program-only (avaliar antes de prometer feature parity entre as 3)
- Ordem sugerida: Fresha primeiro (mais clientes Dublin) → Booksy → Square Appointments

**Outros itens Fase 3:**
- [ ] AI Helper (sugestões de campanha automáticas) — reutilizar Claude da Sprint 5
- [ ] Staff app (PWA para validação de stamps)
- [ ] Integração POS (Square, SumUp, Lightspeed) — distinto de Square Appointments; POS é checkout
- [ ] Partner program (agências revendem SmartTap)
- [ ] Expansão: Dublin → Cork → Galway
- [ ] White-label enterprise

**Meta Fase 3 (revisada Fase 3):** 100-180 clientes ativos ao fim do mês 24 → €5.200-8.500 MRR (BASE) / até €13k MRR (otimista, com network effect Dublin).

---

## MÉTRICAS DE SUCESSO (revisado Fase 3 — números defensáveis)

### Business — Cenário BASE
| Métrica | Mês 3 | Mês 6 | Mês 12 | Mês 24 |
|---|---|---|---|---|
| Clientes pagantes | 5 | 15 | 38 | 104 |
| MRR | €145 | €570 | €1.748 | €5.200 |
| ARR run-rate | €1.740 | €6.840 | €20.976 | €62.400 |
| Setup fees acum | €98 | €784 | €2.058 | €5.733 |
| Churn mensal | 0% (pilotos) | 0% | 6% | 5% |

### Cenário CONSERVADOR (50% do BASE) — fallback
- Mês 24: ~52 clientes, MRR ~€2.200, ARR ~€26k → negócio como side-project

### Cenário OTIMISTA (network effect Dublin + WoM forte)
- Mês 24: ~180-200 clientes, MRR ~€13.000, ARR ~€156k → momento de contratar SDR/CS

### Break-even
- **Operacional** (sem salário fundador): Mês 3
- **Com salário fundador €3k/mês:** Mês 21-22 (BASE)
- **Runway necessária:** ~€25k gap salarial em 12-18 meses → cobrir com Palmeiras Dublin + freelance parcial

### Produto (indicadores de saúde)
- Taps por stand por dia: **meta >5**
- Tap → Review: **meta >20%**
- Tap → Stamp: **meta >40%**
- Retorno em 30 dias: **meta >30%**

### North Star Metric
**Cliente com >50 membros cadastrados = produto funcionando**

---

## REGRAS DO PROJETO (SIGA SEMPRE)

1. **Stack é fixo** — não trocar tecnologias sem justificativa muito forte
2. **Identidade visual é sagrada** — não mudar cores, fontes, logo sem aprovação
3. **MVP é MVP** — não adicionar features não listadas sem aprovação
4. **Monorepo sempre** — todo código novo vai para o package correto
5. **RLS obrigatório** — todo acesso ao banco passa por Row Level Security
6. **GDPR por design** — todo dado de consumidor precisa de consent explícito
7. **Mobile-first** — toda interface começa pensando no celular
8. **Testes antes de código** — TDD para lógica de negócio crítica
9. **Sem segredos no código** — toda key vai para .env
10. **Documentar decisões** — toda decisão técnica importante vai para ADR

---

## ESTADO ATUAL DO PROJETO

**Data:** Maio 2026
**Fase atual:** 🟢 **PRODUÇÃO LIVE** — Sprint 4 completo, deploy em prod concluído. Aguardando primeiro pilot real.
**Próximo passo:** S5-W0 (NFC tag CRUD UI) — bloqueador identificado: sem UI de tags, fechar o primeiro cliente exige SQL manual.

**O que está LIVE em produção:**
- **Frontend:** https://smarttap.ie (Vercel, apex + www redirect)
- **Backend:** https://api.smarttap.ie (Railway, Dockerfile + uv + uvicorn)
- **DB:** Supabase prod `qmemsvkeiygdwxyzadrc` (eu-west-1) — 6 migrations aplicadas, RLS ativo em 10 tabelas
- **Stripe:** live mode com 8 price IDs (4 recurring + 4 setup) + webhook validado
- **Resend:** smarttap.ie verificado, emails entregando
- **Cron:** cron-job.org agendado (daily reactivation + monthly report dia 1)
- **Observabilidade:** Sentry (backend + web), Vercel Analytics
- **Runbook completo:** `DEPLOY.md` na raiz do repo

**Sprints completos:**
- Sprint 0 ✅ Setup monorepo + CI/CD
- Sprint 1 ✅ NFC tap loop + customer view + GDPR opt-in
- Sprint 2 ✅ Auth + dashboard + customers + reward config + settings + redeem + CSV export
- Sprint 3 ✅ Stripe (checkout + webhook + billing UI + trial enforcement + transactional emails)
- Sprint 4 ✅ Campaigns (double-stamp + reactivation + monthly PDF report + customer segmentation)

**O que NÃO foi exercitado em prod ainda (smoke tests deferidos):**
- Signup → onboarding → dashboard com pagamento real
- NFC tap → stamp → reward em hardware físico
- Stripe checkout completando + ativando plano via webhook
- Deferidos intencionalmente para o primeiro pilot real (não desperdiçar €78 num teste sintético)

**Operacional:**
- Nenhum cliente pagante ainda
- Pendente: primeira NFC tag impressa + primeiro stand entregue
- Founding member offer pronta (stand grátis + 60d grátis + €29/mês vitalício pros 5 primeiros)

---

## COMO TRABALHAR NESTE PROJETO

Quando retomar uma sessão, comece sempre com:
```
Lê o CLAUDE.md e me diz em qual sprint estamos e qual é o próximo passo.
```

**Cadência atual** (sprints incrementais, pequenos, com aprovação por escopo):
1. Propor escopo da sprint/feature antes de codar (lista de entregáveis + decisões abertas)
2. Henrique aprova ou ajusta
3. Implementar com commits atômicos por sub-fase
4. Lint/typecheck/tests verdes antes de cada commit
5. Push direto pra `main` (sem PR — single dev) — deploy automático: Vercel + Railway auto-redeployam

**Para reverter um deploy ruim:**
- Railway: Deployments → previous green → "Redeploy"
- Vercel: Deployments → "Promote to Production" na última green
- Rollback completo no `DEPLOY.md` §"Rollback playbook"

**Para tarefas específicas (skills gstack):**
- `/office-hours` — revisão executiva antes de decisão estratégica
- `/review` — code review antes de commit grande
- `/qa` — QA completo antes de mudança em produção
- `/cso` — auditoria de segurança

---
> Source: [cotah/SmartTap](https://github.com/cotah/SmartTap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
