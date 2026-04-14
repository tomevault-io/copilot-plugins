## gdrive-downloader

> Ferramenta para baixar arquivos e pastas públicos do Google Drive, com interface gráfica (GUI) e linha de comando (CLI). Desenvolvida para usuários não técnicos no Windows.

# Google Drive Downloader

Ferramenta para baixar arquivos e pastas públicos do Google Drive, com interface gráfica (GUI) e linha de comando (CLI). Desenvolvida para usuários não técnicos no Windows.

## Estrutura do projeto

```
automacao_google_drive/
├── main.py                     # Ponto de entrada: --gui abre GUI, sem flag usa CLI
├── requirements.txt            # Dependências: gdown, tqdm, colorama
├── guia-de-uso.md              # Documentação para o usuário final (não técnico)
├── logs/                       # Logs gerados automaticamente em gdrive_downloader.log
└── gdrive_downloader/          # Pacote principal
    ├── __init__.py             # Exporta apenas download()
    ├── core.py                 # Função central download() — usada por CLI e GUI
    ├── api_downloader.py       # Download via Google Drive API v3 (necessário para pastas)
    ├── cli.py                  # Interface de linha de comando (argparse)
    ├── gui.py                  # Interface gráfica (tkinter)
    ├── config.py               # Persistência de configurações em ~/.config/gdrive_downloader/config.json
    ├── logger.py               # Logger com arquivo em logs/gdrive_downloader.log
    ├── progress.py             # CLIProgressReporter e GUIProgressReporter
    └── utils.py                # extract_id_from_url(), sanitize_output_path()
```

## Como executar

```bash
# Ativar ambiente virtual (Windows)
venv\Scripts\activate

# Interface gráfica
python main.py --gui

# Linha de comando — arquivo
python main.py "LINK_DO_DRIVE" -o "PASTA_DESTINO"

# Linha de comando — pasta (requer API Key)
python main.py "LINK_DA_PASTA" -o "PASTA_DESTINO" --api-key "AIzaSy..."

# Opções extras
--speed 512     # limita velocidade em KB/s
--resume        # retoma download interrompido
--no-color      # remove cores do terminal
```

## Instalar dependências

```bash
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

## Arquitetura e decisões importantes

- **Função central `core.download()`** é a única interface pública do pacote. CLI e GUI chamam ela da mesma forma, passando um `progress_reporter` diferente.
- **Dois modos de download para pastas:**
  - Com `api_key`: usa Google Drive API v3 via `api_downloader.py` — confiável
  - Sem `api_key`: usa `gdown.download_folder()` — pode ser bloqueado pelo Google
- **Arquivos individuais** funcionam sem API Key via `gdown.download()`
- **API Key** é salva automaticamente em `~/.config/gdrive_downloader/config.json` via `config.py`
- **GUI** usa `quiet=True` no gdown (progresso vem do `GUIProgressReporter` por file watching)
- **CLI** deixa o tqdm do gdown aparecer diretamente no terminal

## Tecnologias

- Python 3.8+
- `gdown` — download de arquivos/pastas do Google Drive
- `tqdm` — barra de progresso no terminal
- `colorama` — cores no terminal Windows
- `tkinter` — interface gráfica (stdlib do Python)
- Google Drive API v3 (via requests, sem SDK oficial)

## Configuração persistida

A API Key do Google é salva em `~/.config/gdrive_downloader/config.json`. Não há arquivo `.env` — a chave fica apenas neste JSON do usuário.

## Público-alvo

Usuários finais não técnicos no Windows. O `guia-de-uso.md` é o manual deles — deve ser mantido em linguagem simples e sem jargão técnico.

## Fluxo de trabalho com Git

Ao finalizar qualquer tarefa que altere código (fix, feat, refactor, chore, docs, etc.), fazer commit na branch atual com uma mensagem descritiva da tarefa realizada.

Formato da mensagem de commit:
```
<tipo>: <descrição curta do que foi feito>

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

Tipos aceitos: `feat`, `fix`, `refactor`, `chore`, `docs`, `style`, `test`

Exemplo:
```
feat: adiciona suporte a download com limite de velocidade

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

Nunca pular hooks (`--no-verify`) nem forçar push sem confirmação explícita do usuário.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danielfranca47)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/danielfranca47)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
