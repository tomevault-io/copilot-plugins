## kiroku

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Kiroku

App web pessoal (usuário único) para gerenciar animes e leituras (manga/manhwa/manhua).

## Governança

A **constituição** do projeto está em [`.specify/memory/constitution.md`](.specify/memory/constitution.md)
— 10 princípios não-negociáveis. Ela supersede qualquer hábito de código.

Portão antes do merge: `typecheck` + `lint` + `test` + `build`, todos com zero erro.

## Documentação viva por área

Cada área-chave tem seu próprio `CLAUDE.md`, descrito por responsabilidade. Mudou regra,
endpoint, padrão ou estrutura? Atualize o doc da área **no mesmo PR** — doc desatualizada é bug.

Listadas na direção do fluxo: **schema → service → rota → UI** (Princípio II).

| Área | O que vive lá |
|------|---------------|
| [`prisma/`](prisma/CLAUDE.md) | Schema e convenções Prisma + MongoDB (`db push`, ObjectId, composite types) |
| [`lib/services/auth/`](lib/services/auth/CLAUDE.md) | Regra pura de autenticação (allowlist do dono) |
| [`lib/services/works/`](lib/services/works/CLAUDE.md) | Regra e persistência de obras: adicionar com cópia, dedupe, editar catálogo |
| [`lib/services/entries/`](lib/services/entries/CLAUDE.md) | Regra e persistência do acompanhamento: status, progresso, histórico, drop, revejo |
| [`lib/services/organization/`](lib/services/organization/CLAUDE.md) | Eixos da coleção: listas e tags — as duas regras de tag, de que lado mora cada associação |
| [`lib/services/collection/`](lib/services/collection/CLAUDE.md) | O recorte da coleção: busca, filtros e ordenação — regra pura, 0 escrita, o recorte mora no endereço |
| [`lib/services/stats/`](lib/services/stats/CLAUDE.md) | A visão de cima: estatísticas e recomendações — agregação pura, 0 escrita, 0 cache, dado ausente declarado |
| [`lib/media/`](lib/media/CLAUDE.md) | O que varia por `mediaType`: esquema de progresso, rótulos, os 5 status, rota de detalhe |
| [`lib/appearance/`](lib/appearance/CLAUDE.md) | Preferência de aparência (tema + fonte): regra pura fail-soft por eixo, cookie no navegador, 0 schema |
| [`lib/anime-api/`](lib/anime-api/CLAUDE.md) | Adapter da fonte externa (AniList) e o mapeamento para a cópia local |
| [`lib/auth/`](lib/auth/CLAUDE.md) | Infra do Auth.js: sessão, DAL (`verifyOwner`), guard (`requireOwner`) |
| [`lib/http/`](lib/http/CLAUDE.md) | Envelope de resposta, códigos de erro e o handler central das rotas |
| [`app/api/`](app/api/CLAUDE.md) | Route handlers e o envelope de resposta uniforme |
| [`components/`](components/CLAUDE.md) | Primitivos shadcn (`ui/`) e componentes compostos; tokens de aparência |

## Features

Uma pasta por feature em `specs/00N-*`, uma branch por feature.

- [`specs/001-owner-auth/`](specs/001-owner-auth/) — portão de acesso privado do dono.
- [`specs/002-design-system-shell/`](specs/002-design-system-shell/) — base visual (tokens,
  componentes) e casca de navegação da área logada.
- [`specs/003-anime-tracking/`](specs/003-anime-tracking/) — núcleo de acompanhamento de animes:
  adição com cópia dos dados, status/progresso, grid, detalhe, Continue Assistindo e histórico.
- [`specs/004-reading-tracking/`](specs/004-reading-tracking/) — leituras (manga/manhwa/manhua) no
  mesmo motor: cadastro sempre manual, progresso por capítulo contínuo com volume só de exibição,
  vocabulário por tipo de mídia e a tela unificada de retomar.
- [`specs/005-ratings-and-notes/`](specs/005-ratings-and-notes/) — avaliação e anotações ricas: nota
  geral do dono, categorias aplicáveis por tipo, nota por unidade com uma sugerida derivada (nunca
  persistida, nunca aplicada sozinha), três listas estruturadas e histórico com o antes→depois.
- [`specs/006-collection-organization/`](specs/006-collection-organization/) — os eixos da coleção:
  gênero editável nos 4 tipos, tags livres com autocompletar que ignora acento, listas do dono com
  Favoritos garantido, e a regra de que operação global não escreve no histórico das obras.
- [`specs/007-collection-search/`](specs/007-collection-search/) — busca, filtros e ordenação: usar
  os eixos da 006 para recortar a coleção. Feature 100% de leitura — o recorte mora no endereço,
  a regra é pura e vive no servidor, e a busca sem acento roda em memória (o Prisma+Mongo não a faz).
- [`specs/008-stats-dashboard/`](specs/008-stats-dashboard/) — painel de estatísticas e recomendações:
  a visão de cima da coleção. Feature 100% de leitura, 0 cache — dado ausente é sempre declarado,
  episódio nunca funde com capítulo, e a recomendação sai só da coleção do dono (global, piso de 5).
- [`specs/009-visual-theming/`](specs/009-visual-theming/) — temas visuais e movimento: 3 paletas
  escuras × 4 fontes de título, eixos ortogonais escolhidos pelo dono, lembrados por cookie e
  aplicados no 1º paint (sem flash). Reverte a decisão dark-only da 002 — ainda sem next-themes.

---
> Source: [DevysonSilva/kiroku](https://github.com/DevysonSilva/kiroku) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
