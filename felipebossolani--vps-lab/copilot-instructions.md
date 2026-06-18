## vps-lab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Visão geral

Repositório público da série **vps-lab** — documentando a construção de infraestrutura self-hosted com VPS + Coolify. Cada episódio (`01-security/`, `02-coolify/`, etc.) contém scripts prontos para rodar, checklists e docs técnicos.

Repo complementar privado: [vps-lab-content](https://github.com/felipebossolani/vps-lab-content) (rascunhos de posts, prints, controle editorial).

---

## Dois repositórios

| Repo | Visibilidade | Conteúdo |
|------|-------------|----------|
| [vps-lab](https://github.com/felipebossolani/vps-lab) | Público | Scripts, código, checklists, docs técnicos |
| [vps-lab-content](https://github.com/felipebossolani/vps-lab-content) | Privado | Rascunhos de posts, prints, controle editorial |

---

## Contexto

- **Autor:** Felipe Bossolani
- **Provider VPS:** Bork Cloud (nacional)
- **OS:** Ubuntu 24.04
- **PaaS:** Coolify (self-hosted)
- **Domínio:** `vpslab.com.br`
- **Usuário SSH:** `deploy` (root bloqueado via SSH)
- **Stack:** Docker, Node.js 20, PostgreSQL 16, Astro

## Comandos de desenvolvimento

**API encurtador de URLs (05-backend-with-db/example-api/):**
```bash
npm run dev          # inicia com --watch
npm start            # produção
docker compose up    # stack completa (API + PostgreSQL)
```

**Site Astro (03-static-sites/example-astro/):**
```bash
npm run dev          # servidor de desenvolvimento
npm run build        # build estático
```

**Blog Astro (04-blog/example-blog/):**
```bash
npm run dev          # servidor de desenvolvimento
npm run build        # build estático (content collections)
```

## Arquitetura

Os scripts de cada episódio são feitos para rodar **no servidor VPS**, não localmente. Subdomínios mapeiam para serviços:

`coolify.` · `site.` · `blog.` · `api.` · `status.` · `grafana.` — todos sob `vpslab.com.br`

A stack de observabilidade (EP06) usa Docker Compose com Grafana, Prometheus, Loki, Promtail, Node Exporter e Uptime Kuma na rede compartilhada `vpslab-monitoring`.

## Convenções de shell scripts

- Sempre iniciar com `set -euo pipefail`
- Scripts devem ser **idempotentes** (seguros para re-executar)
- Output colorido com helpers padronizados: `log()`, `warn()`, `error()`, `section()`
- Nunca incluir IPs reais, senhas ou chaves nos arquivos
- Scripts rodam como root (ou com sudo) em Ubuntu 24.04

## Regras de conteúdo

- `docs/lessons-learned.md` deve ser atualizado após cada episódio concluído
- `README.md` tem tabela de status dos episódios — manter sincronizada
- Status dos episódios: `✅` (concluído), `🔜` (próximo)
- A cada novo episódio, criar também o post correspondente no blog (`04-blog/example-blog/src/content/posts/`)

---
> Source: [felipebossolani/vps-lab](https://github.com/felipebossolani/vps-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
