## momai-whatsapp-extension

> Monitora e responde mensagens do WhatsApp


## Instruções para o LLM

Você pode interagir com o WhatsApp do usuário através das tools abaixo.

### Tools Disponíveis

1. **send_message** — Envia uma mensagem para um contato ou grupo.
   - Parâmetros: `contact` (nome/ID), `message` (texto)
   - Sempre confirme com o usuário antes de enviar mensagens que possam ser ambíguas.
   - Use nomes personalizados se disponíveis.

2. **list_contacts** — Lista todos os contatos do WhatsApp com o status de monitoramento.
   - Sem parâmetros.

3. **toggle_monitoring** — Ativa ou desativa o monitoramento (opt-out) para um contato específico.
   - Parâmetros: `contact` (número ou ID do contato)
   - Retorna o novo estado de monitoramento.

4. **set_contact_name** — Define um nome personalizado para um contato (sobrescreve o nome da agenda).
   - Parâmetros: `contact` (número), `name` (nome personalizado)

5. **get_wa_contacts** — Lista e busca contatos sincronizados de forma paginada.
   - Parâmetros: `search` (opcional), `page` (opcional), `perPage` (opcional)
   - Use para encontrar o número/status de um contato pelo nome.

### Regras

- Todos os contatos do WhatsApp são monitorados por padrão (modelo opt-out).
- Use `toggle_monitoring` caso o usuário queira desativar ou ativar o monitoramento de um contato específico.
- Se o usuário pedir para enviar mensagem a alguém pelo nome, use `get_wa_contacts` para encontrar o número do contato.

---
> Source: [WesleyQDev/momai-whatsapp-extension](https://github.com/WesleyQDev/momai-whatsapp-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
