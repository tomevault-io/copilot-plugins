## claude-pixelbar

> > Lido automaticamente ao entrar neste projeto. **Não busque arquivos — use a tabela abaixo.**

# CLAUDE.md — claude-statusline

> Lido automaticamente ao entrar neste projeto. **Não busque arquivos — use a tabela abaixo.**

## O que é

Status bar animada para o Claude Code com: pixel art sprites, dead zone awareness (verde/amarelo/vermelho), geração de temas via IA, e theme picker visual (web UI na porta 7337).

Repositório público: `https://github.com/meresiam/claude-statusline`  
MeresBrain: `4. Projetos/Pessoais/claude-statusline/_MOC Claude Statusline.md`

---

## Mapa de arquivos (leia direto — não glob, não grep)

| Arquivo | Papel |
|---|---|
| `statusline.js` | Hook principal — roda como status bar no Claude Code. Lógica de animação, dead zone, tokens, custo. |
| `statusline-server.js` | Servidor HTTP na porta **7337** — serve a UI do theme picker e theme studio. Entry point pra qualquer trabalho de UI. |
| `generate-theme.js` | Gerador de tema via Claude Haiku — recebe descrição em texto, devolve JSON de tema e ativa. |
| `theme-studio.js` | Studio avançado de edição de temas (interface visual). |
| `install.sh` | Installer Mac/Linux — copia `statusline.js` e configura hooks Claude Code. |
| `install.ps1` | Installer Windows (PowerShell). |
| `README.md` | Documentação pública (temas, uso, API de temas manuais). |

**Arquivos de config do usuário (fora do projeto, em `~/.claude/`):**
- `statusline-config.json` — tema ativo e configuração corrente
- `statusline-themes.json` — store de temas customizados (base + gerados)
- `statusline-server.pid` — PID do servidor quando rodando

---

## Roteamento por intenção

| O Meres pede | Onde mexer |
|---|---|
| "melhora a UI", "interface do server", "tema picker" | `statusline-server.js` |
| "novo tema", "gera tema X", "generator" | `generate-theme.js` |
| "dead zone", "context bar", "tokens", "custo", "spinner" | `statusline.js` |
| "instalar em outro lugar", "installer" | `install.sh` / `install.ps1` |
| "theme studio", "editor de tema visual" | `theme-studio.js` |

---

## Como testar

- Servidor UI: `node statusline-server.js` → abre `http://localhost:7337`
- Gerar tema: `node generate-theme.js "Barbie"` (requer `ANTHROPIC_API_KEY`)
- Ver temas: `node generate-theme.js --list`
- Playwright smoke test: navegar em `http://localhost:7337` e verificar que theme picker carrega

---

## Regras críticas

1. **Porta 7337** — não mudar sem atualizar `statusline.js` também (ele detecta o server).
2. **Temas base são read-only** — `BASE_THEMES` em `statusline-server.js` nunca devem ser sobrescritos, só adicionados via `statusline-themes.json`.
3. **Claude Haiku** — o gerador usa `claude-haiku-4-5-20251001` por custo. Não trocar pra Sonnet.
4. **ANSI 256 colors** — pixel art usa índices 0–255 ANSI, não hex. Ver tabela no README.

---

## Status atual (26-04-2026)

Fase: Funcionalidade completa. Quando um tema é selecionado no picker (UI ou API), o VS Code inteiro sincroniza automaticamente.

**Novo: VS Code sync** — `setActive(name)` chama `applyVSCodeTheme(name)`, que escreve em `~/.../Code/User/settings.json`. Todos os 10 temas base têm paletas definidas em `VSCODE_BASE_COLORS`. Temas customizados (gerados por IA) derivam primary=accent, secondary=darken(accent, 0.65).

Próxima ação: commit + push para o GitHub (para que `install.ps1` baixe a versão nova).

---
> Source: [meresiam/claude-pixelbar](https://github.com/meresiam/claude-pixelbar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
