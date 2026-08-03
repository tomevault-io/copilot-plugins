## mkt-afiliado

> Máquina de busca e divulgação de promoções com links de afiliados.

# CLAUDE.md — mkt-afiliado

Máquina de busca e divulgação de promoções com links de afiliados.
Canais de distribuição: grupos de WhatsApp, TikTok e Instagram.
Dono: Sene (CoHub). Este arquivo é a memória do projeto — mantenha-o atualizado a cada decisão relevante.

## Arquitetura (pipeline)

COLETA → CURADORIA (score) → AFILIAÇÃO (deep link + slug) → CRIATIVO (copy/card) → DISTRIBUIÇÃO → MÉTRICAS

- **n8n** (wf2.cohub.com.br): orquestra coleta, score, criativo e disparo
- **Supabase** (fonte de verdade): projeto `mbmysbiygkcxyuqpdjnc`
- **promo-dash** (Vercel): dashboard de resultados + encurtador de links
- **Evolution API** (node-cohub-01): disparo nos grupos de WhatsApp

## Estado atual (jul/2026)

### ✅ Feito
1. **Schema Supabase aplicado** via migration `mkt_afiliado_schema_inicial` no projeto `mbmysbiygkcxyuqpdjnc`:
   - Tabelas: `channels`, `platforms` (5 pré-cadastradas), `offers`, `links`, `clicks`, `conversions`, `posts`
   - Views: `v_resumo_diario`, `v_performance_oferta`, `v_performance_canal`
   - RLS: SELECT público em tudo; INSERT em `clicks` liberado pro anon (usado pelo redirect edge); demais escritas exigem service_role
   - `conversions` tem unique(platform_id, pedido_externo) → upsert idempotente
2. **apps/promo-dash** — Next.js 14 (app router), build validado:
   - `/` visão geral (KPIs: cliques hoje/7d/30d; pedidos, receita, comissão pendente/aprovada 30d; top ofertas)
   - `/ofertas` e `/canais` (views de performance com taxa de conversão)
   - `/importar` — upload CSV de conversões (Amazon/Meli, plataformas sem API de comissão); mapeamento de colunas em `app/importar/page.tsx`, upsert em `app/api/importar/route.ts`
   - `/r/[slug]` — encurtador edge: busca `links` por slug, loga clique (user_agent, referer, ip_hash SHA-256 truncado), redirect 302
   - Basic auth via env `DASH_PASSWORD` (middleware.ts); rotas `/r/*` sempre públicas
   - Design: estética "encarte de ofertas" — papel #f6f5f2, tinta #17181c, amarelo tarja #ffd23f, vermelho preço #d7263d, verde comissão #1e9e5a; fontes Archivo (display) + Inter (corpo). Sem Tailwind, CSS puro em globals.css

### ⚠️ Pendente
1. **Deploy na Vercel** (team CoHUB, `team_U1BoOoBySgAf79a0y3QvvysS`):
   - Importar repo, Root Directory: `apps/promo-dash`
   - Env vars: `SUPABASE_SERVICE_ROLE_KEY` (Supabase → Settings → API) e `DASH_PASSWORD`
   - URL/publishable key do Supabase têm fallback hardcoded em `lib/supabase.ts` (são públicas por design)
   - Obs: a integração Vercel↔Claude.ai não tinha permissão de criar projetos (escopo restrito)
2. **Workflows n8n** (não criados ainda): coleta Shopee → `offers`; geração de `links` (slug + sub_id); disparo WhatsApp via Evolution; sync de conversões a cada 6h (Shopee GraphQL API)
3. **Definições em aberto**: quais programas de afiliados já estão aprovados; grupos WhatsApp próprios vs de terceiros

## Decisões estratégicas (não regredir)

- **Atribuição clique→venda**: passar o slug do encurtador como `sub_id` no link de afiliado (Shopee, AliExpress e Awin suportam) → conversão volta casada com o canal exato. Amazon/Meli: atribuição só no nível da oferta + import CSV manual
- **Plataformas por automação**: Shopee/Ali/Awin = 100% API; Amazon/Meli = CSV via /importar
- **Prioridade de entrada**: Shopee primeiro (aprovação ~1 dia, melhor API do mercado BR)
- **WhatsApp anti-ban** (aprendizado do caso EuSAUDE): chip dedicado (NUNCA o principal), aquecimento 2-3 semanas, máx 20-30 envios/dia por instância no início, intervalo randômico entre grupos, variação de texto por envio. Cloud API oficial NÃO suporta grupos — é Baileys com disciplina
- **Compliance**: toda divulgação leva identificação de publicidade (#publi / "link de afiliado") — exigência CONAR e das plataformas; Amazon derruba conta por omissão
- **Encurtador próprio** (não bit.ly): tracking por grupo/canal é o dado mais valioso da operação

## Infra de referência
- n8n: wf2.cohub.com.br (MCP server disponível)
- Evolution API: stack Docker Swarm em node-cohub-01, gerência via Portainer (mudanças de stack SEMPRE pelo editor do Portainer). API key via $AUTHENTICATION_API_KEY, nunca hardcoded
- Supabase project ref: mbmysbiygkcxyuqpdjnc

## Convenções
- Código e comentários em pt-BR; commits em pt-BR estilo conventional (feat:, fix:, docs:)
- Nunca commitar service_role key ou API keys — só via env vars
- Escritas no banco pelo n8n usam service_role; o dashboard só lê

---
> Source: [julianosene8888/mkt-afiliado](https://github.com/julianosene8888/mkt-afiliado) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
