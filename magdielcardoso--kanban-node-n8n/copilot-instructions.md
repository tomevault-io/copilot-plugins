## kanban-node-n8n

> Esta regra orienta a criação de um Community Node n8n para integração entre Kanban e Chatwoot, seguindo os padrões oficiais da comunidade n8n.

# n8n Community Node: Kanban para Chatwoot

## Objetivo
Esta regra orienta a criação de um Community Node n8n para integração entre Kanban e Chatwoot, seguindo os padrões oficiais da comunidade n8n.

## Padrões Obrigatórios
- O pacote **deve** ser publicado no npm com nome iniciando por `n8n-nodes-` ou `@<scope>/n8n-nodes-` (ex: `n8n-nodes-kanban-chatwoot`).
- O campo `keywords` do `package.json` **deve** incluir `n8n-community-node-package`.
- O arquivo `package.json` **deve** conter o atributo `n8n` listando os nodes e credenciais exportados.
- O node **deve** ser validado localmente e passar pelo linter do n8n.
- O node **deve** ser testado localmente antes de publicação.
- O pacote **deve** ser publicado no npm para ser aceito na comunidade.

## Estrutura Recomendada
- Use o [starter node oficial](mdc:https:/github.com/n8n-io/n8n-nodes-starter) como base.
- Organize o código em diretórios claros: `nodes/`, `credentials/`, `tests/`.
- Documente o node no README do pacote.

## Referências
- [Documentação oficial de Community Nodes](mdc:https:/docs.n8n.io/integrations/community-nodes/build-community-nodes)
- [Exemplo de starter node](mdc:https:/github.com/n8n-io/n8n-nodes-starter)

## Checklist para submissão
- [ ] Nome do pacote conforme padrão
- [ ] `n8n-community-node-package` em `keywords`
- [ ] Atributo `n8n` no `package.json`
- [ ] Lint e testes locais executados
- [ ] Publicado no npm

---
Para dúvidas ou atualizações, consulte sempre a documentação oficial do n8n.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/magdielcardoso) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
