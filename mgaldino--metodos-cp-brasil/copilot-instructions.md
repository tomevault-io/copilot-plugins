## metodos-cp-brasil

> - O objetivo substantivo do paper é classificar o corpus completo elegível de artigos SciELO 2005-2025 para analisar a revolução da credibilidade na Ciência Política brasileira.

# Instruções para Agentes

## Objetivo do Repositório

- O objetivo substantivo do paper é classificar o corpus completo elegível de artigos SciELO 2005-2025 para analisar a revolução da credibilidade na Ciência Política brasileira.
- Os 208 artigos classificados até aqui, reduzidos a 175 após exclusões, são apenas amostra de validação/piloto. Eles servem para calibrar, auditar e validar o schema de classificação; não são a base final de análise substantiva do paper.
- `Brazilian Journal of Political Economy` e `Civitas - Revista de Ciências Sociais` não entram na análise principal. Seus registros podem permanecer preservados em dados brutos e artefatos rastreáveis, mas devem ser excluídos de qualquer base analítica substantiva.

## Fluxo de Trabalho

- Quem revisa não implementa; quem implementa não revisa o próprio trabalho.
- Revisões devem produzir relatório, não editar arquivos.
- Implementações que alterem dados, scripts ou relatórios devem ser revisadas por agente independente antes de commit/push quando houver risco metodológico ou reprodutível.
- Não ampliar escopo sem pedido explícito. Se a tarefa é validar, validar e relatar; não aplicar decisões, criar overrides ou refatorar pipeline.

## Skills e Revisão

- Usar skills quando apropriado, especialmente `review-r`, `data-analysis-r`, `review-python` e `devils-advocate`.
- Usar `review-r` para revisar scripts R e artefatos derivados de scripts R.
- Usar `data-analysis-r` para análises e validações de dados em R.
- Usar `review-python` para revisar scripts Python.
- Usar `devils-advocate` quando a decisão metodológica, interpretação ou escopo puder comprometer a validade do paper.

## Reprodutibilidade

- Análises em R devem estar em scripts, não em comandos inline.
- Todo output derivado precisa ter script de geração. Se não há script, não há auditoria; se não há auditoria, não é reprodutível.
- Preservar dados brutos e snapshots usados em validação, com fonte e data de acesso documentadas.
- Usar `dplyr::select` ao selecionar colunas em R.

---
> Source: [mgaldino/metodos-cp-brasil](https://github.com/mgaldino/metodos-cp-brasil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
