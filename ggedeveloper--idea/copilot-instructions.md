## idea

> REGRAS LOCAIS DO PROJETO PARA O CASCADE


REGRAS LOCAIS DO PROJETO PARA O CASCADE

1. Princípios Gerais
- Seguir sempre as regras do projeto (ver RASCUNHO_RULES_PROJETO.md).
- Garantir rastreabilidade, transparência e documentação de todas as ações 
relevantes.

2. Estrutura de Registo e Indexação Obrigatória

2.1. Erros e Resolução de Erros
- Todo erro detectado ou resolvido deve ser registrado com:
  - Timestamp (data/hora UTC)
  - Tipo (erro, warning, resolved)
  - Descrição detalhada
  - Stack trace (quando aplicável)
  - Ação tomada
  - Indexação: ID único sequencial
- Os registros devem ser mantidos em LOG_ERROS.md ou sistema equivalente.

2.2. Prompts
- Todos os prompts relevantes (decisões, dúvidas, interações importantes 
com o usuário) devem ser registrados com:
  - Timestamp
  - Tipo (prompt, decisão, dúvida, resposta)
  - Conteúdo do prompt
  - Resultado/decisão
  - Indexação: ID único sequencial
- Os registros podem ser mantidos em LOG_PROMPTS.md.

2.3. Implementações, Atualizações e Detalhes Técnicos
- Toda implementação de código, atualização relevante, decisão técnica ou 
alteração de arquitetura deve ser registrada com:
  - Timestamp
  - Tipo (implementação, atualização, refactor, decisão técnica)
  - Descrição detalhada
  - Arquivos/áreas afetadas
  - Indexação: ID único sequencial
- Os registros podem ser mantidos em LOG_CODE.md ou documentação técnica 
específica (docs/ ou similar).

3. Organização dos Registos
- Os logs devem ser facilmente pesquisáveis (por data, ID, tipo).
- Recomenda-se manter um índice inicial em cada ficheiro de log.
- Sempre registrar em tempo real ou imediatamente após cada ação relevante.

---

## Consulta Obrigatória da Documentação e Dados

- O agente Cascade deve SEMPRE consultar:
  - O índice de documentação (`docs/INDEX_DOCS.md`)
  - Todos os documentos referenciados no índice
  - Os ficheiros de dados e integração relevantes:
    - `data/api/info-geko-api-users.txt` (instruções, chaves, links da API Geko)
    - `data/xml/geko_full_en_utf8.xml` (XML completo dos produtos)
- Antes de:
  - Tomar decisões técnicas
  - Implementar código
  - Realizar integrações com API ou parsing de dados
  - Responder a prompts relevantes
- A consulta constante a estes recursos é obrigatória para garantir alinhamento, rastreabilidade, máxima qualidade e integração correta dos dados.

4. Boas Práticas para o Cascade
- Validar sempre a existência de logs antes de criar entradas duplicadas.
- Garantir que todos os timestamps estejam em UTC.
- Garantir que logs de erros não exponham dados sensíveis.
- Atualizar este documento sempre que novas regras ou necessidades surgirem.

Última atualização: 2025-06-04

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GGEDeveloper) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
