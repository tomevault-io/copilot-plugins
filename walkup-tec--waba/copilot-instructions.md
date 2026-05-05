## ai-task-archive-doc

> Toda tarefa gera log detalhado em doc/ (memória do projeto)


# Regras para bom funcionamento das IAs (operacional)

## Objetivo

Evitar duplicação e retrabalho, criando uma memória detalhada do projeto em `doc/`.

## Como trabalhar em cada pedido

1. Antes de executar qualquer ação, pensar rapidamente na forma mais **eficiente e rápida** de chegar ao resultado com qualidade.
2. Verificar se o tema já existe em `doc/`:
   - Se houver um log anterior com solução semelhante, reaproveitar e apenas complementar o que mudou.

## Ao executar uma tarefa (sempre que eu rodar comandos e/ou fizer mudanças no repositório)

1. Garantir que existe a pasta `doc/` na raiz do projeto.
2. Criar um arquivo markdown detalhado em `doc/` com nome:
   - `doc/LOG-YYYY-MM-DD__HHmmss__<slug-da-tarefa>.md`
   - Regra do slug:
     - usar apenas letras minúsculas, números e `-`
     - remover caracteres especiais e acentos
     - substituir espaços por `-`
     - limitar a um máximo de 60 caracteres
     - prefira incluir palavras-chave do tema (ex.: `dockerfile`, `n8n-webhook`, `supabase-migration`, `integracao-apis`)
     - incluir o “tipo” quando fizer sentido (ex.: `create`, `update`, `fix`, `refactor`)
3. O arquivo deve conter, no mínimo:
   - Contexto do pedido (resumo)
   - Comandos solicitados pelo usuário (se houver) e/ou ações executadas por mim (se o usuário não solicitou comandos)
   - Solução implementada (passo a passo)
   - Arquivos criados/alterados
   - Como validar (testes/rodar/comandos de verificação) quando aplicável
   - Observações de segurança (segredos não expostos, configs, etc.)
   - Itens para evitar duplicação no futuro (palavras-chave)
4. Atualizar a memória consolidada em `doc/memoria.md` com um resumo do que foi feito e quais palavras-chave pesquisar na próxima vez.

---
> Source: [walkup-tec/waba](https://github.com/walkup-tec/waba) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
