## sao-joao-arcoverde

> Instruções para agentes de IA trabalhando neste repositório.

# CLAUDE.md

Instruções para agentes de IA trabalhando neste repositório.

## Leitura obrigatória no início de cada sessão

todas as vezes que você fizer um bugfix faça um teste de regressão

## Regras obrigatórias

- Sempre escrever em **português (PT-BR)**: comentários, mensagens de commit, strings de UI.
- Nunca remover funcionalidades existentes sem autorização explícita.
- Nunca instalar dependências novas sem perguntar primeiro.
- Sempre perguntar antes de refatorar arquivos que não foram mencionados na tarefa.

### Mobile-first
- Projetar para 375px de largura mínima.
- Testar mentalmente o layout em tela pequena antes de sugerir qualquer componente.
- Evitar tabelas, layouts complexos ou elementos que quebram em mobile.

---

## O que NÃO fazer

- ❌ Usar Google Maps embed — usar link clicável `maps.google.com/?q=` no lugar.
- ❌ Criar backend ou API routes no MVP.
- ❌ Adicionar autenticação.
- ❌ Usar `localStorage` diretamente — usar Dexie para persistência local.
- ❌ Usar `console.log` em código de produção.

---

## Fluxo esperado para cada tarefa

1. Ler a tarefa e identificar quais arquivos serão criados ou modificados.
2. Checar se já existe algo relacionado no projeto antes de criar do zero.
3. Criar dados em `src/lib/data/` se a página precisar de conteúdo.
4. Criar componentes reutilizáveis em `src/components/` se necessário.
5. Criar ou editar a rota em `src/routes/`.
6. Informar quais arquivos foram criados/modificados ao final.

---
## Contexto de desenvolvimento

- Desenvolvedor solo, iniciante em programação.
- Prefere explicações simples junto com o código.
- Foco em MVP — evitar over-engineering.

---
> Source: [mnzcarvalho/sao-joao-arcoverde](https://github.com/mnzcarvalho/sao-joao-arcoverde) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
