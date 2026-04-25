## hector

> Porteiro final — NUNCA concluir tarefa sem executar deploy se houve alterações deployáveis. Ver deploy-after-changes.mdc.


# Porteiro: concluir só após deploy

Antes de dar qualquer tarefa por **concluída**:

- Se houve alterações em código, config, docs ou scripts deployáveis → **obrigatório** executar `git add -A`, `git commit`, `git push origin main` antes de dizer que terminou.
- Se ainda não fez o push, **não conclua** — faça o deploy agora.
- Exceção única: utilizador disse explicitamente "não quero deploy" ou é rascunho/experimento.

**Falhar = não considerar a tarefa concluída.** Esta regra existe para garantir que o deploy nunca seja esquecido.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gabrielhectorimob) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
