## docker-deployment

> Configuracao Docker e deploy via Coolify


# Docker e Deploy (Coolify)

## Servicos Docker Compose
- **app**: Laravel + PHP 8.3-FPM + Nginx
- **worker**: Laravel Horizon (filas)
- **scheduler**: Laravel Scheduler (cron)
- **reverb**: Laravel Reverb (WebSocket)
- **mysql**: MySQL 8.0
- **redis**: Redis 7 Alpine
- **minio**: MinIO (storage S3-compativel)
- **meilisearch**: Meilisearch (busca full-text)

## Variaveis de Ambiente
- Chaves de API de IA ficam em `.env` (OPENAI_API_KEY, GEMINI_API_KEY, ANTHROPIC_API_KEY)
- Credenciais de redes sociais ficam em `.env`
- NUNCA commitar `.env` - usar `.env.example` como template
- Coolify injeta variaveis via interface

## Volumes Persistentes
- MySQL: `mysql_data:/var/lib/mysql`
- MinIO: `minio_data:/data`
- Redis: `redis_data:/data`
- Meilisearch: `meilisearch_data:/meili_data`

## Health Checks
- Todos os servicos devem ter health check configurado
- App: `curl -f http://localhost/health`
- MySQL: `mysqladmin ping`
- Redis: `redis-cli ping`

## Build
- Multi-stage build para otimizar imagem
- Composer install com `--no-dev` em producao
- npm run build para assets Vue
- Cachear config, routes e views em producao

---
> Source: [julioandolfo/mkt-privus](https://github.com/julioandolfo/mkt-privus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
