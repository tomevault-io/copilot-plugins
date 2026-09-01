## agente-zap-pix

> Agente de WhatsApp (qualificador + agendador) — FastAPI + Gemini + Supabase/pgvector.

# CLAUDE.md — guia do projeto

Agente de WhatsApp (qualificador + agendador) — FastAPI + Gemini + Supabase/pgvector.
Visão geral em `README.md`; spec e plano em `docs/superpowers/`.

## Comandos (Windows)
- Testes: `.venv/Scripts/python -m pytest -v`  (NÃO use `pytest` puro)
- Rodar servidor: `uvicorn app.main:app --port 8000`
- Ingestão: `.venv/Scripts/python -m ingest.ingest ./conhecimento --reset`
- Scripts soltos que importam `app` precisam de cwd = raiz do repo (o `.env` é lido do cwd).

## Arquitetura
- `app/main.py` (webhook) → responde 200 rápido → background `process_event` → `app/orchestrator.py`.
- `orchestrator.handle_message` (o cérebro): por turno faz RAG (`rag.retrieve`) + **uma** chamada estruturada ao Gemini (`gemini_client.generate_turn` → `TurnResult`), mescla o lead, classifica, persiste (`store.py`) e envia (`whatsapp.send_text`).
- RAG: `gemini-embedding-001` (768 dims, L2) → `store.search_documents` (RPC `match_documents`, distância de cosseno).
- Saída estruturada do Gemini = modelo pydantic `TurnResult` (em `app/models.py`).

## Convenções / cuidados
- `Classificacao.etiqueta` é `Literal["quente","morno","frio"]` (casa com o `CHECK` de `leads.etiqueta`).
- Calendly: o bot **oferece** o link no momento certo e só envia com o **consentimento do cliente** (acao `mandar_calendly` + nome/necessidade presentes), **uma vez** por conversa (guard `already_scheduled` via `conv.lead_id`); **reenvio** só quando o cliente pede explicitamente (ação `reenviar_link` do LLM + o flag `link_ja_enviado` vai no prompt via `build_user_turn`).
- `ParsedMessage.msg_type`: `"text"` segue o fluxo normal; **áudio é transcrito** (`whatsapp.download_media` → `gemini_client.transcribe_audio` → vira turno de texto; falha → fallback educado); outras mídias recebem fallback educado; **reações são ignoradas** (`parse_incoming` → None). OGG/Opus do WhatsApp validado em produção.
- `whatsapp.mark_read_and_typing(message_id)`: ticks azuis + "digitando..." (1 POST; best-effort no `process_event`, falha não bloqueia o turno).
- Prompt de naturalidade: persona Pix Safety, 1 pergunta por vez, anti-repetição, nome do perfil do WhatsApp (`contact_name`) usado pra confirmar identidade; `temperature=0.6` no `generate_turn`.
- A função do store é `insert_document` (foi renomeada de `upsert_document`).
- Config via env/.env com pydantic-settings; use `get_settings()` (cacheado), não instancie `Settings()` solto fora de testes.
- Segredos só no `.env` (gitignored) — nunca no `.env.example`.
- TDD: cada feature tem teste; mantenha a suíte verde antes de commitar.

## Estado atual
**EM PRODUÇÃO** — o bot está ao vivo no WhatsApp (+55 51 2391-7020), validado com leads
reais: texto, **voice notes transcritas** (OGG/Opus OK), qualificação, agendamento com
consentimento e reenvio sob pedido. **64 testes passando**, em `main` (GitHub
`Rasantis/agente_zap_pix`, autodeploy no Render). Base real do Pix Safety indexada (26 chunks).
Erros de produção ficam em `error_logs` no Supabase (consultar via MCP `supabase`).
Cuidado: schema wire sem defaults (`TurnResultWire`) é obrigatório — a API do Gemini/SDK
rejeita defaults no `response_schema`; não voltar a passar `TurnResult` direto.
Pendências operacionais (antes de escalar tráfego): planos pagos Render/Supabase/Gemini e
rotação de segredos — checklist em `docs/superpowers/plans/2026-07-08-melhorias-pos-golive.md`
(a transcrição de áudio do plano JÁ foi implementada). Auditoria de versões com upgrade
recomendado (não aplicado): `notas/relatorio-auditoria-versoes.md`. Contexto na memória do projeto.

---
> Source: [Rasantis/agente_zap_pix](https://github.com/Rasantis/agente_zap_pix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
