## orquestrador-maestro

> Este repositório é um espelho público e sanitizado de uma configuração Orquestrador/Codex. Ele deve continuar instalável, revisável e seguro para publicação.

# AGENTS.md

## Escopo

Este repositório é um espelho público e sanitizado de uma configuração Orquestrador/Codex. Ele deve continuar instalável, revisável e seguro para publicação.

## Regras De Edição

- Não inclua dados pessoais, caminhos reais de usuário, tokens, logs, backups, memórias locais ou arquivos de cache.
- Prefira alterar a fonte local em `~/.orquestrador` e depois rodar `scripts/sync-from-local.ps1`.
- Depois de qualquer sync ou edição, rode `scripts/validate-public.ps1`.
- Não faça commit nem push automaticamente.
- Trate `DEV/` neste clone como memória operacional local ignorada pelo Git; publique a convenção em `docs/` e scripts, não o worklog local.
- Use UTF-8 e corrija texto quebrado ou mojibake antes de concluir.

## Arquivos Gerados

- `orquestrador/` é snapshot exportado e sanitizado.
- `codex/` contém skills, agentes e prompts Codex exportados de forma sanitizada.
- `skill-library/community-skills/` contém a biblioteca deduplicada de skills.
- `tool-profiles/` contém hooks e perfis textuais selecionados.
- `home/AGENTS.md` é o contrato global que será instalado no home do usuário.
- `.local/` é privado, ignorado pelo Git e pode guardar listas locais de termos privados para sanitização.

## Publicação

Antes de subir:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\validate-public.ps1
git diff -- .
```

---
> Source: [FernandoBolzan/Orquestrador-Maestro](https://github.com/FernandoBolzan/Orquestrador-Maestro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
