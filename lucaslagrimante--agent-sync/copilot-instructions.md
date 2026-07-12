## agent-sync

> Script Python que escaneia um diretório com múltiplos projetos e padroniza as configurações de ferramentas de IA via symlinks canônicos.

# agents-sync

Script Python que escaneia um diretório com múltiplos projetos e padroniza as configurações de ferramentas de IA via symlinks canônicos.

## Como funciona

Execute na raiz do diretório pai (ex: `Projects/`):

```bash
python sync_agents.py
```

O script escaneia todos os subdiretórios, exibe uma tabela de conformidade e deixa o usuário escolher quais projetos padronizar.

## Estrutura canônica aplicada

```
repo/
├── AGENTS.md                          ← canônico (Codex/OpenAI)
├── CLAUDE.md                          → AGENTS.md
├── GEMINI.md                          → AGENTS.md
├── .cursorrules                       → AGENTS.md
├── .windsurfrules                     → AGENTS.md
├── .github/
│   ├── copilot-instructions.md        → ../AGENTS.md
│   ├── agents/                        → ../.agents/agents   (custom agents .agent.md)
│   └── prompts/                       → ../.agents/workflows (prompt files)
├── .gemini/system.md                  → ../AGENTS.md
├── .agents/
│   ├── workflows/                     (pasta real — slash commands)
│   ├── skills/                        (pasta real — skills)
│   ├── agents/                        (pasta real — custom agents .agent.md)
│   └── commands                       → workflows  (alias)
├── .claude/
│   ├── commands                       → ../.agents/workflows
│   ├── skills                         → ../.agents/skills
│   └── agents/                        → ../.agents/agents
└── .opencode/
    ├── commands                       → ../.agents/workflows
    └── skills                         → ../.agents/skills
```

## Dependência

```bash
pip install rich
```

## Regras para agentes

- Este é um projeto de utilitário Python simples — mantenha tudo em `sync_agents.py`
- Não adicione dependências além de `rich`
- O script deve funcionar em Windows, macOS e Linux
- No Windows, symlinks de diretório exigem Developer Mode ou execução como Administrador
- O script é idempotente — pode ser executado múltiplas vezes sem efeitos colaterais
- Não sobrescreva arquivos regulares no lugar de symlinks — apenas avise o usuário

---
> Source: [LucasLagrimante/agent-sync](https://github.com/LucasLagrimante/agent-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
