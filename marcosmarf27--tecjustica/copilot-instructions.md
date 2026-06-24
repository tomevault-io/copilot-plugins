## tecjustica

> Instrucoes do projeto para o Claude Code. Leia antes de editar.

# CLAUDE.md — skills-tecjustica

Instrucoes do projeto para o Claude Code. Leia antes de editar.

## Regra critica de publicacao: SEMPRE bumpe a versao

Este repo e distribuido como plugin do Claude Code via marketplace. O Claude
Code indexa o cache de cada instalacao por versao, em:

```
~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/
```

Se voce mexe em qualquer skill, no `.mcp.json`, ou em qualquer coisa que o
plugin carrega, **voce TEM que bumpar a versao em dois arquivos antes de
commitar e dar push**:

1. `.claude-plugin/plugin.json` → campo `"version"`
2. `.claude-plugin/marketplace.json` → campo `"version"` dentro do bloco do
   plugin em `plugins[0].version`

Os dois precisam ficar iguais.

### Por que? (Historia real — abril/2026)

Entre marco e abril foram adicionadas 5 skills novas (`tecjustica-mcp-lite`,
`tecjustica-parse`, `tecjustica-docx`, `pje-download`, `cjf-jurisprudencia`) e
o MCP foi ligado no `.mcp.json`, mas a versao continuou `1.0.0`. Resultado:
**todos os usuarios que instalaram antes de marco ficaram travados vendo
apenas 2 skills (`analise-processo-civil` e `analise-processo-penal`) e sem
MCP conectado**, porque:

- `/plugin marketplace update` re-clona o marketplace
- `/plugin install` le a versao declarada → ve `1.0.0` → compara com o cache
  local → "ja instalado, nada a fazer"
- O cache `cache/.../tecjustica/1.0.0/` continua congelado no snapshot
  antigo, com 2 skills e um `.mcp.json` velho

Para usuarios completamente novos funciona, porque o snapshot `1.0.0` e
criado do zero. Mas para qualquer um que ja tinha instalado, a atualizacao
nunca chega.

Isso nao e bug do Claude Code — e como semver sempre funcionou. Republicar
conteudo sob a mesma versao e mentir pros clientes que nada mudou.

### Convencao semver para este repo

- `patch` (1.1.0 → 1.1.1): correcao de bug numa skill existente, edicao de
  prompt, ajuste de script bundled, fix de script.
- `minor` (1.1.0 → 1.2.0): skill nova, tool MCP nova, campo novo no frontmatter,
  dependencia nova — qualquer coisa aditiva.
- `major` (1.x → 2.0.0): remove ou renomeia skill/tool, muda schema do MCP,
  remove dependencia obrigatoria — qualquer coisa que quebre usuarios.

### Checklist antes de `git push`

- [ ] Alterei alguma skill, script bundled, `.mcp.json`, ou `plugin.json`?
- [ ] Se sim, decidi se e patch/minor/major e bumpei em **ambos** os arquivos
      `.claude-plugin/plugin.json` e `.claude-plugin/marketplace.json`.
- [ ] Commit inclui a mensagem `chore: bump plugin version to X.Y.Z` quando
      for so o bump, ou referencia o bump no corpo quando for parte de outro
      commit.

### Instrucao para os usuarios apos um bump

```
/plugin marketplace update tecjustica-plugins
/plugin install tecjustica@tecjustica-plugins
```

O Claude detecta versao nova, cria `cache/.../tecjustica/X.Y.Z/` fresco com
todo o conteudo atual, e troca o plugin ativo. Nao precisa limpar cache
manualmente, nao precisa mexer em `installed_plugins.json`, nao precisa
reclonar nada.

## Estrutura do plugin

- `.claude-plugin/plugin.json` — manifesto do plugin (nome, versao, paths)
- `.claude-plugin/marketplace.json` — manifesto do marketplace (declara o
  plugin)
- `.mcp.json` — registro do servidor MCP TecJustica Lite (carregado
  automaticamente quando o plugin e habilitado)
- `skills/<nome-skill>/SKILL.md` — cada skill model-invoked
- `skills/<nome-skill>/scripts/` — scripts bundled referenciados via
  `${CLAUDE_SKILL_DIR}`
- `install.sh` — instalador de dependencias de sistema (Node, Chrome,
  browser-use, LibreOffice, python-docx, fontes editoriais)

## Variaveis de ambiente esperadas

- `TECJUSTICA_API_KEY` (prefixo `mcp_`) — lida pelo `.mcp.json` via
  `Authorization: Bearer ${TECJUSTICA_API_KEY}`. Obrigatoria para o MCP.
- `TECJUSTICA_PARSE_API_KEY` (prefixo `tjp_`) — lida pelo `scripts/parse.sh`
  da skill `tecjustica-parse` como header `X-API-Key`. Opcional (so OCR).

Ambas sao lidas do ambiente do terminal em que `claude` foi iniciado. Se
faltar uma, o erro aparece em runtime (MCP 401, ou parse.sh reclama).

---
> Source: [marcosmarf27/tecjustica](https://github.com/marcosmarf27/tecjustica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
