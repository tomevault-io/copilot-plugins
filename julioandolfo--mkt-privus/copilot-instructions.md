## project-overview

> Visao geral do projeto MKT Privus - plataforma interna de marketing com IA


# MKT Privus - Plataforma de Marketing com IA

## Sobre o Projeto
Plataforma interna de marketing com IA para gestao de 4-10 marcas/empresas. NAO e um SaaS.
Inspirada no BestContent.ai, mas com funcionalidades extras de analytics e metricas customizadas.
Deploy via Docker no Coolify.

## Stack Tecnologica
- **Backend**: Laravel 12 (PHP 8.3)
- **Frontend**: Vue 3 (Composition API) + Inertia.js 2.0
- **UI**: Tailwind CSS 4 + Shadcn-vue
- **Database**: MySQL 8.0
- **Cache/Filas**: Redis 7 + Laravel Horizon
- **WebSocket**: Laravel Reverb
- **Storage**: MinIO (S3-compativel)
- **Busca**: Meilisearch
- **Graficos**: Apache ECharts
- **Editor Rich-Text**: TipTap

## Modulos Principais
1. **Core**: Gestao de marcas (multi-tenant por brand_id)
2. **Social**: Criacao de conteudo com IA + agendamento + publicacao (Instagram, Facebook, LinkedIn, TikTok, YouTube, Pinterest)
3. **Chat**: Assistente IA multi-modelo (OpenAI, Gemini, Claude)
4. **Blog**: Geracao de artigos SEO + integracao WordPress
5. **Links**: Paginas de bio link
6. **Analytics Hub**: Integracao GA4, Google Ads, Meta Ads, TikTok Ads
7. **Metricas Custom**: Metricas dinamicas com input manual e graficos comparativos
8. **Autopilot**: Automacao de conteudo

## Idioma
- Codigo (variaveis, classes, metodos): ingles
- Comentarios e documentacao: portugues Brasil
- Interface do usuario (UI): portugues Brasil
- Commits: portugues Brasil

---
> Source: [julioandolfo/mkt-privus](https://github.com/julioandolfo/mkt-privus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
