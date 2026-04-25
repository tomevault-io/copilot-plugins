## get-commit-stats

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Sobre este Projeto

CLI Node.js para calcular estatísticas de commits do GitHub. Busca commits diretamente via API do GitHub usando a CLI `gh` e exibe métricas de adições/deleções.

## Pré-requisitos

- **Node.js** (v18+)
- **GitHub CLI (`gh`)** - deve estar instalado e autenticado (`gh auth login`)
- **jq** - usado para parsing de JSON na CLI do GitHub

## Comandos

### Instalar dependências
```bash
npm install
```

### Executar o CLI
```bash
node get_commit_stats.js <usuario> -i <data_inicio> [opcoes]
```

**Argumentos:**
- `usuario` (obrigatório): Nome do usuário do GitHub

**Opções:**
- `-i, --inicio <data>` (obrigatório): Data de início no formato YYYY-MM-DD
- `-f, --fim <data>` (opcional): Data final no formato YYYY-MM-DD (padrão: data atual)
- `-o, --org <organizacao>` (opcional): Filtrar busca por organização
- `-t, --tipo-data <tipo>` (opcional): Tipo de data para filtrar (`author` ou `committer`, padrão: `committer`)
- `-V, --version`: Mostrar versão
- `-h, --help`: Mostrar ajuda

### Exemplos de uso

```bash
# Buscar commits com período específico
node get_commit_stats.js usuario-exemplo --inicio 2026-01-01 --fim 2026-01-31

# Buscar até hoje (data atual como padrão)
node get_commit_stats.js usuario-exemplo --inicio 2026-01-01

# Buscar filtrando por organização
node get_commit_stats.js usuario-exemplo --inicio 2026-01-01 --org minha-organizacao

# Usar formas abreviadas
node get_commit_stats.js usuario-exemplo -i 2026-01-01 -o minha-organizacao
```

## Arquitetura

- **`get_commit_stats.js`**: Script CLI principal usando Commander.js

### Funções de Validação
- `validarUsuario(usuario)`: Valida nome de usuário GitHub (max 39 chars, alfanuméricos/_/-)
- `validarOrganizacao(org)`: Valida nome de organização GitHub (max 39 chars, alfanuméricos/-)
- `validarFormatoData(dataStr)`: Valida formato YYYY-MM-DD
- `validarTipoData(tipoData)`: Valida tipo de data (`author` ou `committer`)
- `sanitizar(input)`: Sanitiza strings para segurança (remove caracteres especiais)

### Classes
- `ProgressBar(total, label)`: Barra de progresso visual com ETA
  - `update(increment, extraInfo)`: Atualiza progresso
  - `setCurrent(current, extraInfo)`: Define valor atual
  - `complete(message)`: Marca como concluído
  - `render(extraInfo)`: Renderiza barra visual

### Funções de Busca
- `buildQuery(usuario, inicio, fim, org, tipoData)`: Constrói query de busca para API
- `buscarRepositoriosDaOrganizacao(org)`: Lista todos os repositórios de uma organização
- `buscarBranchesDoRepositorio(repo)`: Lista branches via API do GitHub, priorizando branches principais (main, master, develop, stage, staging, prod, production, release, hotfix, test, qa)
- `buscarCommitsNoRepositorio(repo, usuario, inicio, fim, tipoData)`: Busca commits em um repositório específico com deduplicação por SHA
- `buscarCommitsPorOrganizacao(usuario, inicio, fim, org, tipoData)`: Busca commits em todos os repos de uma org
- `buscarCommits(usuario, inicio, fim, org, tipoData)`: Função principal que escolhe a estratégia (org vs search)
- `getCommitStats(repo, sha)`: Busca estatísticas de um commit específico via `gh api`

### Funções de Output
- `exibirResultados(results, totalCommits, totalAdditions, totalDeletions, usuario, inicio, fim)`: Imprime relatório formatado
- `main()`: Orquestra o fluxo: parse args → buscar commits → processar → exibir

## Fluxo de Execução

1. **Parse argumentos** via Commander.js (valida datas, define padrões)
2. **Escolha da estratégia de busca:**
   - **Com organização**: Usa `buscarCommitsPorOrganizacao()` (itera todos os repos)
   - **Sem organização**: Usa `gh api search/commits` (limitado a 1000 resultados)
3. **Busca de commits** com filtros (usuário, datas, org, tipo-data)
4. **Dedução** de commits duplicados por SHA
5. **Ordenação** por data decrescente (quando busca por organização)
6. **Agrupamento** por repositório
7. **Para cada commit**, chama `gh api` para obter `stats.additions` e `stats.deletions`
8. **Agregação** de totais por repositório e geral
9. **Impressão** do relatório com formatação de tabela (pt-BR)

### Estratégias de Busca

#### Sem Organização (API Search)
```
gh api "search/commits?q=author:USER+committer-date:START..END+sort:committer-date-desc"
```
- **Vantagem:** Rápido, usa API indexada
- **Limitação:** Máximo 1000 resultados
- **Uso:** Para buscas globais ou períodos pequenos

#### Com Organização (Iteração de Repos)
```
1. gh api "orgs/ORG/repos?per_page=100" (paginado)
2. Para cada repo: gh api "repos/ORG/REPO/commits?author=USER&since=START&until=END"
3. Deduplica commits por SHA
4. Ordena por data decrescente
```
- **Vantagem:** Sem limite de resultados, busca completa
- **Custo:** Mais lento, uma chamada API por repo
- **Uso:** Para relatórios completos de uma organização

## Dependências

- **Node.js** (built-in modules: `child_process`)
- **commander** (^12.0.0): Parser de argumentos CLI
- **GitHub CLI (`gh`)**: deve estar instalado e autenticado
- **`jq`**: usado para parsing de JSON na CLI do GitHub

## Notas

### Segurança
- Todas as entradas são validadas e sanitizadas antes de usar em comandos shell
- `sanitizar()` remove caracteres perigosos para prevenir command injection
- Validação de regex para usuários/orgs segue padrões do GitHub

### Performance
- Erros na chamada da API de estatísticas retornam `{ additions: 0, deletions: 0 }` (falha silenciosa por commit)
- Logs de progresso são escritos em `stderr` para não interferir no output (pode redirecionar com `2>/dev/null`)
- Commits são deduplicados por SHA para evitar contagem dupla
- Barras de progresso mostram ETA e informações extras para melhor UX

### Limites da API
- **GitHub Search API:** Max 100 resultados/página, 1000 totais
- **GitHub Commits API:** Max 100 commits/página
- **Organizações:** Limite de segurança de 1000 repositórios

### Formatação
- Output usa formatação pt-BR (números com separador de milhar)
- Validação de datas usa formato YYYY-MM-DD (ISO 8601)
- Barras de progresso usam caracteres ASCII para compatibilidade

### Tipos de Data
- **author-date:** Data em que o commit foi originalmente criado
- **committer-date:** Data em que o commit foi aplicado ao branch (merge/rebase)
- Padrão: `committer` (mais comum para relatórios de entrega)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ihagofsantos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
