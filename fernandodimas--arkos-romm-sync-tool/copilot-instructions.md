## arkos-romm-sync-tool

> Script bash para sincronização de ROMs e saves entre consoles ArkOS (ARM/Debian) e servidor RomM self-hospedado. Usa `dialog` para TUI, `gptokeyb` para gamepad, API RomM via curl.

# AGENTS.md - RomM-Sync-Tool

## Projeto
Script bash para sincronização de ROMs e saves entre consoles ArkOS (ARM/Debian) e servidor RomM self-hospedado. Usa `dialog` para TUI, `gptokeyb` para gamepad, API RomM via curl.

## Arquivos Principais
- `RomMSync.sh` — Script principal (~1800 linhas)
- `rommsync_updater.sh` — Updater com migrações encadeadas
- `themes/` — Temas de cores `.dialogrc`
- `rommsync.controls` — Mapeamento de controles gptokeyb

## Config
- `~/.rommsync.conf` — Credenciais Base64 Basic Auth (chmod 600)
- `CONF_VERSION="1.4.0"` — Versionamento de config

## Comandos Essenciais
```bash
# Syntax check (RODE SEMPRE APÓS EDITAR)
bash -n RomMSync.sh

# Executar no console
/roms/tools/RomMSync.sh

# Force refresh cache
/roms/tools/RomMSync.sh --force-refresh
```

## Convenções de Código
- `set -euo pipefail` ativo — funções com `|| true` em pipelines que podem falhar
- Variáveis: `ROMM_URL`, `ROMM_USER`, `ROMM_AUTH_B64`, `TMP_DIR`, `CACHE_DIR`
- Cache em `/dev/shm/rommsync/cache/` (RAM) com fallback `/tmp/rommsync`
- `guarantee_tmp()` detecta `/dev/shm` no início
- `DLG_W`/`DLG_H` calculados via `stty size` em `main()`
- `dialog` sempre escreve para `$CURR_TTY` (não stdout)
- `api_get()` usa cache-first strategy; endpoints de escrita bustam cache
- `force_refresh_cli` detecta `--force-refresh` na CLI

## Estrutura do Menu Principal (9 opções)
1. Backup de Saves
2. Download de Jogos
3. Reconfigurar Conexão
4. Status da Conexão (health check expandido)
5. Ver Log
6. Tema de Cores
7. Limpar Cache
8. Atualizar Script
9. Sair

## Temas
4 temas em `themes/`:
- `rommsync_arkos.dialogrc` — Padrão (preto/verde)
- `rommsync_high_contrast.dialogrc` — Alto contraste (amarelo/preto)
- `rommsync_blue.dialogrc` — Azul
- `rommsync_green.dialogrc` — Verde retro

Selecionado via menu e persistido em `DIALOGRC_THEME` no config.

## Updater
`rommsync_updater.sh` com migrações encadeadas:
- Detecta versão anterior via backup ou config
- Migra v1.0.0 → v1.1.0 → v1.2.0 → v1.3.0 → v1.4.0 → v1.4.1
- Chama `show_changelog()` pós-migração
- `self_update()` e `check_update()` passam `NEW_VERSION`, `SCRIPT_DIR`

## Compatibilidade
- ArkOS (ARM/Debian)
- `bash` 4.x+
- `dialog`, `jq`, `curl`, `wget`, `zip`, `bc`
- `gptokeyb` para gamepad (já integrado)

---
> Source: [fernandodimas/ArkOS_RomM-Sync-Tool](https://github.com/fernandodimas/ArkOS_RomM-Sync-Tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
