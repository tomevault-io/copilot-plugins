## church-tools-iasd

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Church Util** é um aplicativo Electron para sonoplastia em igrejas. Gerencia ações, vídeos Provai e Vede, Informativos e recursos multimídia para cultos.

## Setup

```bash
npm install
npm run dev
```

## Development Commands

- `npm run dev` - Inicia app em modo desenvolvimento
- `npm run build` - Cria executável para distribuição
- `npm run build:win` - Build específico para Windows

## Architecture

### Estrutura
- **main/** - Processo principal Electron (Node.js)
  - `index.js` - Entry point, IPC handlers, cron jobs
  - `database.js` - Persistência JSON local
  - `downloads.js` - Downloads, parsing HTML, extração ZIP
- **preload/** - Ponte segura entre main e renderer
- **renderer/** - Interface (HTML/CSS/JS puro)

### Segurança
- Context Isolation ativado
- Node Integration desativado no renderer
- IPC seguro via contextBridge

### Persistência
- JSON em `userData/db.json`
- Cache de 6h para listas de vídeos
- Dias padrão: sábado, domingo, quarta

### Categorias de Ações
1. **Geral**: Arquivos locais diversos
2. **Provai e Vede**: Auto-lista e baixa vídeos do trimestre
3. **Informativo**: Verifica/baixa semanalmente (sábados 18:00)

### Multi-monitor
- Cronômetro fullscreen configurável
- Tela preta fullscreen configurável

## Timezone
Todas as operações usam **America/Sao_Paulo** (UTC-3)

---
> Source: [SergioSilva77/church-tools-iasd](https://github.com/SergioSilva77/church-tools-iasd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
