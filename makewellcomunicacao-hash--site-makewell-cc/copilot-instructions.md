## site-makewell-cc

> Instruções e contexto para o assistente Codex neste projeto.

# AGENTS.md — Makewell Comunicação

Instruções e contexto para o assistente Codex neste projeto.

## Sobre o Projeto

Site institucional da **Makewell Comunicação** — agência de marketing digital.
- **Domínio:** makewell.com.br
- **Repositório:** GitHub → GitHub Pages (deploy automático no push para `main`)
- **Stack:** HTML + CSS + JS vanilla, arquivo único `index.html`
- **Sem frameworks, sem bundlers, sem dependências externas** (exceto GTM e Meta Pixel)

## Regras Obrigatórias

### A cada publicação (git push):
1. **Sempre atualizar `sitemap.xml`** com `<lastmod>` do dia atual
2. Incluir o sitemap no mesmo commit das alterações

### Nunca fazer:
- Adicionar bibliotecas externas sem aprovação explícita do usuário
- Usar `!important` desnecessariamente
- Criar arquivos novos sem necessidade
- Quebrar SEO ou Lighthouse score

## Rastreamento Instalado

- **Google Tag Manager:** GTM-NFGD8X8
- **Meta Pixel:** ID `2571294026447618` — eventos `Lead` nos botões de CTA
- **Schema.org JSON-LD:** MarketingAgency

## Decisões de Design

- **Fontes:** System fonts (`'Segoe UI', system-ui, -apple-system`) — sem Google Fonts para performance
- **Cores:** `--gold: #F07820`, `--gold-light: #F5A623`, `--dark: #080808`
- **Hero h1:** `clamp(38px, 5vw, 72px)` desktop / `clamp(36px, 9vw, 52px)` mobile
- **Hero h1 em:** `display: block`, `padding-bottom: 0.15em` (evita corte nos descenders)
- **Hero-left:** `overflow: hidden; min-width: 0` (evita overflow em Windows/Segoe UI)
- **Grain texture:** `body::after` com SVG noise, `will-change: transform` (sem `mix-blend-mode` — causa scroll lag)
- **Hero card:** glassmorphism com `backdrop-filter: blur(24px)`
- **Menu mobile:** `justify-content: flex-start; padding-top: 100px`

## Endereço

R. Prof. Lourenço Peixoto, 36, Sala 04, Jatiúca, Maceió, AL.

## Informações da Empresa

- CNPJ: 27.238.877/0001-71
- WhatsApp: +55 82 99910-3246
- E-mail: contato@makewell.com.br
- Fundação: 2017 (São Paulo/SP). Expansão para o Nordeste em 2020, com sede em Maceió (AL) e operação estratégica em São Paulo (SP).

## Deploy

```bash
git add . && git commit -m "mensagem" && git push origin main
```
GitHub Pages faz o deploy automático em 1-3 minutos.

---
> Source: [makewellcomunicacao-hash/site-makewell-cc](https://github.com/makewellcomunicacao-hash/site-makewell-cc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
