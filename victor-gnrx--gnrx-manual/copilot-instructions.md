## gnrx-manual

> Manual de usuário do gnrx (GitBook), organizado em `epi/`, `formularios/`, `auditorias/`, etc., cada um com seu próprio `SUMMARY.md`.

# gnrx-manual

Manual de usuário do gnrx (GitBook), organizado em `epi/`, `formularios/`, `auditorias/`, etc., cada um com seu próprio `SUMMARY.md`.

## Público-alvo

Este manual é para o **cliente final** (usuário do sistema e, no caso de integrações, o desenvolvedor externo que consome a API/webhooks do gnrx) — não para desenvolvedores internos do gnrx.

Ao documentar features de integração (webhooks, APIs):
- Documente apenas o que o cliente final precisa fazer/saber (ex: como receber e validar um webhook, payloads de eventos).
- Não inclua endpoints ou fluxos internos usados pelo frontend do gnrx para configurar a feature (ex: API de CRUD que só a tela interna de Integrações chama) — isso não é relevante para quem consome a integração.

---
> Source: [victor-gnrx/gnrx-manual](https://github.com/victor-gnrx/gnrx-manual) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
