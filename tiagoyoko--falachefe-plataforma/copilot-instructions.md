## falachefe-plataforma

> Arquitetura do projeto FalaChefe


# Arquitetura do Projeto FalaChefe

## Visão Geral

Plataforma SaaS de **Chat Multiagente via WhatsApp** usando CrewAI.

## Infraestrutura

### Frontend + Backend (Vercel)
- **URL**: https://falachefe.app.br
- **Stack**: Next.js 15 + React 19 + TypeScript
- **Database**: Supabase PostgreSQL
- **Cache**: Upstash Redis (REST API)
- **Auth**: Better Auth + Google OAuth

### API CrewAI (Servidor Hetzner)
- **IP**: 37.27.248.13:8000
- **Stack**: Python 3.12 + Flask + CrewAI
- **Container**: Docker Compose
- **Workers**: Gunicorn (2 workers, 4 threads)
- **Timeout**: 120s

## Fluxo de Mensagens WhatsApp

```
WhatsApp → UAZAPI
    ↓
Webhook: /api/webhook/uaz (Vercel)
    ↓
MessageService: Valida usuário e empresa
    ↓
    ├─ Usuário SEM empresa → Mensagem padrão
    │   "Acesse falachefe.app.br e cadastre sua empresa"
    │
    └─ Usuário COM empresa → MessageRouter
        ↓
        HTTP POST → http://37.27.248.13:8000/process
        ↓
        CrewAI processa (10-60s)
        ↓
        Resposta → UAZAPI → WhatsApp
```

## Arquivos Principais

### Backend (Vercel)
- [src/app/api/webhook/uaz/route.ts](mdc:src/app/api/webhook/uaz/route.ts) - Webhook WhatsApp
- [src/app/api/chat/route.ts](mdc:src/app/api/chat/route.ts) - Chat web
- [src/services/message-service.ts](mdc:src/services/message-service.ts) - Processamento mensagens
- [src/lib/message-routing/message-router.ts](mdc:src/lib/message-routing/message-router.ts) - Roteamento

### CrewAI (Hetzner)
- [crewai-projects/falachefe_crew/api_server.py](mdc:crewai-projects/falachefe_crew/api_server.py) - Flask API
- [crewai-projects/falachefe_crew/webhook_processor.py](mdc:crewai-projects/falachefe_crew/webhook_processor.py) - Processador
- [crewai-projects/falachefe_crew/src/falachefe_crew/crew.py](mdc:crewai-projects/falachefe_crew/src/falachefe_crew/crew.py) - Crew

### Configuração
- [.env.local](mdc:.env.local) - Variáveis de ambiente (local)
- [vercel.json](mdc:vercel.json) - Configuração Vercel
- [crewai-projects/falachefe_crew/.env](mdc:crewai-projects/falachefe_crew/.env) - Env CrewAI

## Tabelas Principais (Supabase)

- `companies` - Empresas clientes
- `user` - Usuários da plataforma (Better Auth)
- `user_onboarding` - Dados de onboarding
- `user_subscriptions` - Relação usuário ↔ empresa
- `conversations` - Conversações WhatsApp
- `messages` - Mensagens
- `agent_memories` - Memória dos agentes

## Domínios

- **falachefe.app.br** → Vercel (Next.js)
- **api.falachefe.app.br** → Hetzner (CrewAI) - ⚠️ DNS não configurado

## Variáveis de Ambiente Críticas

### Vercel (.env.local)
- `CREWAI_API_URL` - URL do servidor CrewAI
- `UAZAPI_TOKEN` - Token WhatsApp
- `POSTGRES_URL` - Database Supabase
- `UPSTASH_REDIS_REST_URL` - Cache Redis

### Hetzner (.env)
- `OPENAI_API_KEY` - API OpenAI
- `UAZAPI_TOKEN` - Token WhatsApp
- `GUNICORN_WORKERS` - Workers (2)
- `GUNICORN_TIMEOUT` - Timeout (120s)

## Deploy

### Vercel (Automático)
```
git push origin master → Vercel detecta → Build → Deploy
```

### Hetzner (Manual via SSH)
```bash
ssh root@37.27.248.13
cd /opt/falachefe-crewai
docker compose up -d
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tiagoyoko)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tiagoyoko)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
