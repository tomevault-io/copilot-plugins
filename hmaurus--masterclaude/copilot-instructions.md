## masterclaude

> Marketplace de plugins para Claude Code.

# Masterclaude - Guidelines de Desenvolvimento

Marketplace de plugins para Claude Code.

## Versionamento

- **Semver**: MAJOR.MINOR.PATCH
- **Quando incrementar**:
  - PATCH (0.0.X): correções de bugs
  - MINOR (0.X.0): novas funcionalidades compatíveis
  - MAJOR (X.0.0): mudanças incompatíveis

### Arquivos a atualizar

| Arquivo | Quando |
|---------|--------|
| `plugins/<plugin>/.claude-plugin/plugin.json` | Sempre que alterar o plugin |
| `.claude-plugin/marketplace.json` (`metadata.version`) | Sempre que publicar alterações |

## Workflow de Desenvolvimento

1. Fazer alterações no plugin
2. Atualizar versão do plugin em `plugins/<plugin>/.claude-plugin/plugin.json`
3. Atualizar versão do marketplace em `.claude-plugin/marketplace.json`
4. Commit e push

## Comandos em Plugins

- Evitar operadores shell compostos (`||`, `&&`, `;`) em comandos inline `!`comando``
- Deixar lógica condicional em texto (Claude interpreta naturalmente)
- Comandos simples são mais compatíveis com CLI e extensão VS Code

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hmaurus)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hmaurus)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
