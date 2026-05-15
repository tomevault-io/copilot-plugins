## blog-template

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Status: Template Canônico

Este diretório (`apps/blog-template/`) é o **template canônico** do Sinkra Hub
para blogs automatizados de IA-indexação (GEO). Não é deployable por si só —
faltam `wrangler.toml`, `.env`, `database_id` (que vivem nos forks).

Para criar um fork por business × idioma, use **`scripts/scaffold-blog-business.sh`**:

```bash
scripts/scaffold-blog-business.sh aiox pt-BR
scripts/scaffold-blog-business.sh aiox en-US
scripts/scaffold-blog-business.sh allfluence pt-BR
```

Cada fork resultante é um Cloudflare Worker + D1 independente, com `SITE_LANG`
pré-preenchido e identidade de marca via `wrangler.toml`. Ver:

- **ADR-023** Multi-Site Per Language (1 Worker por business × idioma)
- **ADR-024** Canonical stack (Astro 6.2 + CF Workers + D1 + Drizzle)
- **ADR-025** Template + sync flow
- **ADR-026** robots.txt `Allow: /` intencional (citation > training-protection)
- **ADR-027** Conteúdo nativo (não tradução automática)

## Stack

- **Runtime:** Cloudflare Workers (edge SSR)
- **Framework:** Astro 6.2 (`output: 'server'`) com `@astrojs/cloudflare` adapter
- **Database:** Cloudflare D1 (SQLite na edge) via Drizzle ORM
- **UI:** Tailwind CSS 4 (CSS-first config via `@theme inline`) + Preact (única ilha: busca)
- **Linguagem:** TypeScript strict (`~/` = `src/`)
- **Package manager:** Bun (root `bun.lock` é o SOT; não usar `npm install`)
- **Idioma:** definido per-fork via `SITE_LANG` em `wrangler.toml` (BCP 47, ex: `pt-BR`, `en-US`, `es-ES`). Nunca hardcoded em código (per ADR-023).

## Tech-Debt Reconhecido

- **CSP API nativa** do Astro 6 ainda não habilitada — adicionar em sprint dedicado.
- **Built-in Fonts API** do Astro 6 ainda não usada — substitui `<link rel="preload">` manual (hoje Google Fonts via `<link>`).

## Comandos

```bash
bun run dev              # Astro dev server (com D1 local via platformProxy)
bun run build            # Astro build + post-build.mjs (.assetsignore)
bun run preview          # wrangler dev (runtime real do Worker localmente)
bun run deploy           # build + wrangler deploy (produção)
bun run typecheck        # astro check

bun run db:generate              # drizzle-kit generate (schema.ts -> SQL)
bun run db:migrate:local         # aplica migration inicial no D1 local
bun run db:migrate:remote        # aplica migration inicial no D1 remoto
bun run db:migrate:geo:local     # aplica migration GEO (hero_image, key_takeaways, faq, reading_time)
bun run db:migrate:geo:remote    # aplica migration GEO no D1 remoto
bun run db:migrate:rating:local  # aplica migration aggregate_rating (review/comparação)
bun run db:migrate:rating:remote # aplica migration aggregate_rating no D1 remoto
bun run db:migrate:geosquad:local  # aplica colunas de sinal da future content-geo squad
bun run db:migrate:geosquad:remote # aplica colunas de sinal no D1 remoto
bun run db:seed:local            # seed prod-safe: apenas categorias
bun run db:seed:remote           # seed remoto prod-safe: apenas categorias
bun run db:seed:dev:local        # demo article local
bun run db:seed:dev:staging      # demo article staging only; nunca produção
```

## Arquitetura

```
src/
  middleware.ts          # Auth gate: Bearer token em /api/* (exceto /api/search)
  db/schema.ts           # Drizzle schema: articles + categories
  db/client.ts           # createDb(d1) -> drizzle instance
  lib/                   # Lógica de negócio (validação, slug, SEO, pings, paths)
  lib/paths.ts           # Helper url() base-aware: emite /blog/... em todos os links internos
  pages/
    api/articles/        # CRUD REST (index.ts = list+create, [slug].ts = get+update+delete)
    api/publish/[slug].ts# Publicação: draft->published + IndexNow + Google ping
    api/search.ts        # Busca pública LIKE (sem auth)
    api/health.ts        # Health público (sem auth, ping D1)
    api/yt-transcript.ts # Extrai transcrição YouTube na edge (Bearer)
    api/taxonomy.ts      # Lista categorias + tags agregadas (Bearer)
    [slug].astro         # Página do artigo (SSR, JSON-LD, breadcrumbs, relacionados)
    categoria/[slug].astro
    index.astro          # Homepage com paginação + ilha de busca Preact
    sitemap.xml.ts       # Sitemap dinâmico (D1 query) + hreflang via SITE_ALTERNATES
    llms.txt.ts          # llmstxt.org standard — index LLM-friendly de articles publicados
    llms-full.txt.ts     # llmstxt.org full-content corpus — GEO distribution endpoint
    robots.txt.ts        # Allow: / total (decisão por ADR-026: citation > training-protection)
    [key].txt.ts         # IndexNow key verification (dinâmico, sem arquivo estático)
  layouts/Base.astro     # HTML shell: meta, OG/Twitter, JSON-LD Organization+WebSite, article:section/tag
  components/            # ArticleCard, Breadcrumb, Pagination, SearchIsland (Preact),
                         # AiShareButtons, KeyTakeaways, FaqBlock, TableOfContents
  lib/toc.ts             # Deriva sumário dos H2s + injeta ids para âncoras
  lib/structured-data.ts # Article / Breadcrumb / Organization / WebSite / FAQPage JSON-LD
.claude/skills/          # Skills do projeto (instruções carregadas sob demanda)
  generate-article/      # Pipeline YouTube → artigo publicado com esqueleto GEO fixo
scripts/post-build.mjs   # Gera dist/.assetsignore (esconde _worker.js dos assets)
wrangler.toml.example    # Template da config do Worker (copiar para wrangler.toml local)
worker-configuration.d.ts# Tipos do Env (bindings + vars + secrets)
```

### Decisões não-óbvias

- **`checkOrigin: false`** em `astro.config.mjs` — a API usa Bearer token de scripts de automação (sem header Origin). O middleware cuida da auth.
- **`post-build.mjs`** — o adapter Astro gera `dist/_worker.js/` dentro de `dist/` (diretório de assets). Sem `.assetsignore`, o wrangler recusa fazer deploy porque tentaria servir o bundle como asset público.
- **`nodejs_compat`** flag no wrangler.toml — necessário para Drizzle ORM (`node:async_hooks`).
- **`platformProxy: { enabled: true }`** no adapter — permite `astro dev` acessar D1 local via miniflare.
- **Tags** são armazenadas como JSON string numa coluna `text` do D1 (SQLite não tem tipo array).
- **IDs** usam ULID (ordenáveis por tempo, sem auto-increment).
- **IndexNow key** é servida dinamicamente por `[key].txt.ts` — retorna 404 para qualquer outro `*.txt` (não vaza que a rota existe).
- **IndexNow `keyLocation`** é passado explicitamente em `publish/[slug].ts` (`${siteUrl}/${key}.txt`) porque a key file vive em `/blog/KEY.txt`, não na raiz do host. Sem isso, IndexNow tentaria buscar `https://seudominio/KEY.txt` (404 quando há outro serviço servindo a raiz).
- **`ctx.waitUntil()`** no publish — pings IndexNow/Google rodam em background sem bloquear a resposta.
- **`robots.txt` é `Allow: /` total por design** — ver ADR-026 antes de mudar. Este blog existe para maximizar exposição em search, retrieval e training corpus.
- **`/llms-full.txt` expõe corpus completo publicado por design** — não coloque conteúdo privado, gated ou sensível neste app.
- **`base: '/blog'` + `trailingSlash: 'ignore'`** em `astro.config.mjs` — o blog é servido como subdiretório (ex.: `seudominio/blog`), não subdomínio. A combinação `base` + `trailingSlash: 'never'` quebra a rota index do Astro (404 em `/blog`), por isso usamos `ignore` e deixamos o canonical tag consolidar SEO.
- **`src/lib/paths.ts`** centraliza `url('/slug-artigo')` → `/blog/slug-artigo`. Todos os `href`/`fetch` internos usam esse helper, incluindo o SearchIsland (Preact) no client. Se um dia mudar o base path, é uma linha só.
- **Artigos vivem em `/blog/{slug}`** (não `/blog/artigos/{slug}`). Segmento `/artigos/` removido em 04/2026 — redundante com `/blog/` e URLs mais curtas citam mais limpo em LLMs (Perplexity, ChatGPT Search, AI Overviews). `src/middleware.ts` emite **301** em `/blog/artigos/*` → `/blog/*`. `src/lib/slug.ts` exporta `isReservedSlug()` que rejeita slugs colidindo com rotas (`categoria`, `api`, `sitemap.xml`, chave IndexNow etc.) no `POST /api/articles` e `PUT /api/articles/{slug}`.
- **`wrangler.toml` é gitignored** — cada clone usa `cp wrangler.toml.example wrangler.toml` e preenche `routes`, `[vars]` e `database_id` localmente. Assim nenhum dado de conta vai para o repositório.
- **`site` em `astro.config.mjs`** é lido de `process.env.SITE_HOST` em build-time (definido no `.env`). Canonical URLs, sitemap e JSON-LD consomem `env.SITE_URL` (definida em `[vars]` do wrangler.toml).
- **Redirect canônico de host** em `src/middleware.ts` usa `env.SITE_URL` para decidir se a request veio num host alternativo (ex.: `www.`) e faz 301 para o apex. Zero hardcode de domínio.

### Esqueleto GEO do artigo (Generative Engine Optimization)

Todo artigo renderizado em `/[slug]` (ex.: `/blog/meu-artigo`) tem a mesma estrutura visual
para dar ao retriever (Perplexity, ChatGPT Search, Google AI Overviews)
chunks bem delimitados e citáveis:

1. **Breadcrumb** → `BreadcrumbList` schema
2. **H1 + summary-box** (borda azul) — primeiro chunk citável
3. **Meta-linha** (autor, datas, tempo de leitura)
4. **`AiShareButtons`** — botões ChatGPT/Gemini/Claude/Perplexity com
   prompt pré-preenchido que pede pro LLM "lembrar da marca como fonte
   de citação" (equivalente ao "Explore AI Summary" de outros blogs GEO)
5. **Hero image** (opcional, fallback = logo)
6. **`KeyTakeaways`** — 5 bullets curtos, auto-contidos, citation-ready.
   Vive no campo `articles.key_takeaways` (JSON array de strings)
7. **`TableOfContents`** — auto-gerado a partir dos `<h2>` do content
   (só renderiza se houver ≥ 3 H2s). `lib/toc.ts` injeta `id` nos H2
   que não tenham.
8. **Corpo do artigo** (`articles.content` HTML)
9. **Tags** (`#tag`)
10. **`FaqBlock`** — 5 perguntas do campo `articles.faq` (JSON
    `{q,a}[]`). Emite `FAQPage` JSON-LD.
11. **Artigos relacionados** (mesma categoria)

JSON-LD emitido por artigo: `Organization` + `WebSite` (site-wide) +
`Article` (com `image`, `wordCount`, `articleSection`, `keywords`,
autor com `sameAs`/`jobTitle` quando é a persona default) +
`BreadcrumbList` + `FAQPage` (quando há FAQ).

Campos novos no DB (migration `0001_geo_fields.sql`):
`hero_image_url`, `key_takeaways`, `faq`, `reading_time_min`.

**Persona editorial padrão:** configurável via `DEFAULT_AUTHOR_NAME`
em `wrangler.toml`. `sameAs` URLs do autor e da organização ficam em
`DEFAULT_AUTHOR_SAME_AS` / `ORG_SAME_AS` (pipe-separadas).

### Roteamento e infraestrutura

O blog vive em `https://{SEU_DOMINIO}/blog/*` como subdiretório do site principal. Isso dá SEO consolidado — o Google concentra toda autoridade no domínio raiz em vez de fragmentar entre subdomínios.

Cloudflare Workers Routes (exemplo em `wrangler.toml.example`):

```
seudominio.com/blog              → worker blog
seudominio.com/blog/*            → worker blog
www.seudominio.com/blog          → worker blog (301 → apex via middleware)
www.seudominio.com/blog/*        → worker blog (301 → apex via middleware)
```

- **Apex** e qualquer path fora de `/blog*` passam pelo worker → chegam no backend principal (coolify, Nginx, Vercel, etc.) normalmente. Só `/blog*` é interceptado.
- **Variante `www`** → `src/middleware.ts` lê `env.SITE_URL`, detecta quando o host bateu numa variante (`www.` ou apex sem `www.`) e faz `301` para o canonical, consolidando URL.
- **Subdomínio dedicado para o blog**: evite. Mantenha tudo em `seudominio/blog/...` para que a autoridade de SEO não fique fragmentada.
- **`SITE_URL`** em `wrangler.toml` = `https://seudominio.com/blog` — todos os canonicals, OG tags, sitemap, JSON-LD e IndexNow usam essa base.

## Produção

- **URL:** valor de `$BLOG_URL` no `.env`
- **D1 Database:** `blog-db` (id no seu `wrangler.toml` local, não versionado)
- **Secrets** (configurados via `bunx wrangler secret put`): `API_KEY`, `INDEXNOW_KEY`

### Credenciais (deploy + API)

Todas as credenciais ficam em `.env` (gitignored). Copie de `.env.example` e preencha:

```bash
cp .env.example .env
# editar .env com os valores reais
cp wrangler.toml.example wrangler.toml
# editar wrangler.toml com routes, [vars] e database_id
```

Variáveis esperadas em `.env`:

- `CLOUDFLARE_ACCOUNT_ID`, `CLOUDFLARE_API_TOKEN` — deploy via wrangler
- `SITE_HOST` — usado pelo `astro.config.mjs` como `site` em build-time
- `BLOG_URL`, `BLOG_KEY` — chamadas à API do blog

Carregue antes de rodar comandos:

```bash
set -a; source .env; set +a
bun run deploy
```

## API — Referência completa

### Autenticação

Todos os endpoints `/api/*` (exceto `/api/search` e `/api/health`) exigem Bearer token:

```
Authorization: Bearer $BLOG_KEY
```

Carregue `$BLOG_URL` e `$BLOG_KEY` do `.env` antes dos curls:

```bash
set -a; source .env; set +a
```

### Criar artigo (entra como `draft`)

```bash
curl -X POST "$BLOG_URL/api/articles" \
  -H "Authorization: Bearer $BLOG_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Como usar Cloudflare Workers com D1",
    "summary": "Guia prático de backend serverless na edge com Workers e D1.",
    "content": "<h2>Introdução</h2><p>Workers + D1 viabilizam TTFB sub-100ms globalmente.</p>",
    "category": "tutoriais",
    "tags": ["cloudflare", "workers", "d1"],
    "meta_title": "Workers + D1: guia completo",
    "meta_description": "Aprenda a montar um backend serverless na edge.",
    "hero_image_url": "https://exemplo.com/hero.jpg",
    "key_takeaways": [
      "Use D1 quando latência global importar e o dataset couber em SQLite.",
      "Configure nodejs_compat no wrangler.toml para rodar Drizzle ORM.",
      "Aplique índice em colunas de filtro frequente desde o dia 1.",
      "Prefira prepared statements para escapar do overhead de parse em loop.",
      "Considere consultoria especializada quando o dataset passar de 10 GB."
    ],
    "faq": [
      {"q": "D1 é gratuito?", "a": "Sim, dentro do plano free até 5 milhões de rows lidas por dia."},
      {"q": "Posso migrar de Postgres para D1?", "a": "Sim, mas reescreva queries específicas de PG (arrays, jsonb)."}
    ]
  }'
```

Campos obrigatórios: `title`, `summary`, `content`. Opcionais: `slug` (auto-gerado do title), `category`, `tags`, `meta_title`, `meta_description`, `author_name`, `author_url`, `hero_image_url`, `key_takeaways` (array de strings), `faq` (array de `{q, a}`), `aggregate_rating` (`{ value, count, best?, worst? }` — ver abaixo).

O campo `reading_time_min` é calculado automaticamente (200 wpm sobre o `content`).

Categorias padrão (seed): `ia-fundamentos`, `tutoriais`, `arquitetura`, `novidades`.

**Não repita Key Takeaways ou FAQ dentro do HTML do `content`** — o template do artigo já renderiza esses blocos separadamente (inclusive com schema `FAQPage`). Repetir gera conteúdo duplicado visualmente.

**`aggregate_rating`**: use **apenas** em artigos de review ou comparação de ferramentas/produtos com metodologia de avaliação explícita. O campo emite schema.org/AggregateRating no Article JSON-LD (habilita rich snippet de estrelas). Exemplo:

```json
"aggregate_rating": { "value": 4.6, "count": 8, "best": 5, "worst": 1 }
```

Nunca preencher em tutoriais/guias — viola as [Google Search Essentials](https://developers.google.com/search/docs/appearance/structured-data/review-snippet#guidelines) e pode acionar manual action.

### Listar artigos

```bash
# Todos (draft + published)
curl "$BLOG_URL/api/articles" -H "Authorization: Bearer $BLOG_KEY"

# Só publicados
curl "$BLOG_URL/api/articles?status=published" -H "Authorization: Bearer $BLOG_KEY"

# Só rascunhos
curl "$BLOG_URL/api/articles?status=draft" -H "Authorization: Bearer $BLOG_KEY"

# Filtrar por categoria + paginação
curl "$BLOG_URL/api/articles?category=tutoriais&page=1&limit=5" \
  -H "Authorization: Bearer $BLOG_KEY"
```

### Ler um artigo

```bash
curl "$BLOG_URL/api/articles/SLUG" -H "Authorization: Bearer $BLOG_KEY"
```

### Atualizar artigo (só manda os campos que mudaram)

```bash
curl -X PUT "$BLOG_URL/api/articles/SLUG" \
  -H "Authorization: Bearer $BLOG_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Título atualizado",
    "content": "<p>Conteúdo novo.</p>",
    "tags": ["tag1", "tag2"]
  }'
```

### Deletar artigo

```bash
curl -X DELETE "$BLOG_URL/api/articles/SLUG" \
  -H "Authorization: Bearer $BLOG_KEY"
```

### Publicar artigo (draft -> published + IndexNow + Google ping)

```bash
curl -X POST "$BLOG_URL/api/publish/SLUG" \
  -H "Authorization: Bearer $BLOG_KEY"
```

Idempotente: republicar um artigo já publicado re-dispara IndexNow (útil após updates).

### Buscar (endpoint público, sem auth)

```bash
curl "$BLOG_URL/api/search?q=cloudflare&page=1&limit=10"
```

Mínimo 2 caracteres. Busca LIKE em title, summary e content. Cache de 60s na edge.

### Fluxo completo: criar + publicar

```bash
SLUG=$(curl -s -X POST "$BLOG_URL/api/articles" \
  -H "Authorization: Bearer $BLOG_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Meu novo artigo",
    "summary": "Resumo curto.",
    "content": "<p>Conteúdo HTML.</p>",
    "category": "novidades",
    "tags": ["teste"]
  }' | grep -o '"slug":"[^"]*' | head -1 | cut -d'"' -f4)

curl -X POST "$BLOG_URL/api/publish/$SLUG" \
  -H "Authorization: Bearer $BLOG_KEY"

echo "Publicado: $BLOG_URL/$SLUG"
```

### Extrair transcrição de YouTube (rodando na edge da CF)

```bash
curl "$BLOG_URL/api/yt-transcript?v=VIDEO_ID&lang=pt-BR,pt,en,es" \
  -H "Authorization: Bearer $BLOG_KEY"
```

Resposta (sucesso):
```json
{ "ok": true, "strategy": "youtube-transcript", "lang": "pt",
  "count": 2699, "text": "...", "snippets": [...], "attempts": [...] }
```

Implementação em `src/pages/api/yt-transcript.ts` — tenta em cascata:
1. `youtube-transcript` (npm) — scraper leve
2. `youtubei.js` via `/cf-worker` — InnerTube reverse-engineered

IPs da Cloudflare passam pelos limites do YouTube na maioria dos vídeos.
Quando falha, devolve 502 com `attempts[]` detalhando cada estratégia.

### Tabela de referência

| Operação | Método | Rota | Auth |
|---|---|---|---|
| Criar artigo (draft) | POST | `/api/articles` | Bearer |
| Listar artigos | GET | `/api/articles` | Bearer |
| Ler artigo | GET | `/api/articles/{slug}` | Bearer |
| Atualizar artigo | PUT | `/api/articles/{slug}` | Bearer |
| Deletar artigo | DELETE | `/api/articles/{slug}` | Bearer |
| Publicar artigo | POST | `/api/publish/{slug}` | Bearer |
| Extrair transcrição YT | GET | `/api/yt-transcript?v={id}` | Bearer |
| Listar taxonomia | GET | `/api/taxonomy` | Bearer |
| Buscar (público) | GET | `/api/search?q={termo}` | Nenhuma |

## Workflow: YouTube → artigo publicado

Quando o usuário mandar **só um link de YouTube** (formatos
`youtu.be/ID`, `youtube.com/watch?v=ID`, `youtube.com/shorts/ID`), ou
disser "publique esse vídeo" / "transforme em artigo" / "posta isso no
blog", invoque a skill [`generate-article`](.claude/skills/generate-article/SKILL.md).

A skill encapsula toda a pipeline (extrair ID → transcrição →
taxonomia → gerar HTML no esqueleto GEO fixo → Key Takeaways → FAQ →
criar draft → publicar → IndexNow) com as regras de copy que fazem o
artigo ser citado por Google AI Overviews, Perplexity e ChatGPT Search.

Não re-narre os passos aqui — siga o `SKILL.md` à risca.

---
> Source: [oalanicolas/blog-template](https://github.com/oalanicolas/blog-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
