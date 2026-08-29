## soofi-model-hosting

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Überblick

Lokales LLM-Setup auf Basis von Docker Compose: ein **Ollama**-Container (Inferenz, GPU-beschleunigt) plus eine **Open WebUI** als Chat-Frontend. Es gibt keinen Anwendungscode — das Repo besteht praktisch nur aus der Orchestrierung (`docker-compose.yml`) und dem persistenten, aber nicht versionierten Ollama-Datenverzeichnis (`ollama/`). Keine Tests, kein Build. Aktuell installiertes Modell: `nemotron-3-nano:30b`.

## Befehle

```bash
# HF_TOKEN frisch laden, BEVOR der Stack startet (für gated HF-Pulls).
# Wird per ${HF_API_KEY} in den ollama-Container durchgereicht, nicht im Repo fixiert.
set -a; . ~/.env.secrets; set +a

# Stack starten / stoppen
docker compose up -d
docker compose down
docker compose logs -f ollama          # Logs verfolgen

# Modelle verwalten (über den laufenden Container)
docker compose exec ollama ollama list
docker compose exec ollama ollama pull nemotron-3-nano:30b
docker compose exec ollama ollama run nemotron-3-nano:30b
docker compose exec ollama ollama pull hf.co/<user>/<repo>:Q4_K_M   # GGUF von HuggingFace

# Eigenes Modell aus einer Modelfile erzeugen.
# files/ ist im Container read-only unter /files gemountet.
docker compose exec ollama ollama create mein-modell -f /files/<name>.modelfile

# API direkt ansprechen (Host-Port 11434)
curl http://localhost:11434/api/tags
```

- Open WebUI: `http://localhost:3000` — Auth ist deaktiviert (`WEBUI_AUTH=false`), reine lokale Nutzung.
- Ollama-API / OpenAI-kompatibler Endpoint: `http://localhost:11434`.

## Architektur

- **Zwei Services in einem Bridge-Netzwerk (`ollama-net`)**: Open WebUI spricht Ollama containerintern über `http://ollama:11434` an (`OLLAMA_BASE_URL`), nicht über den gemappten Host-Port. Wer das Frontend umkonfiguriert, muss diesen internen Namen verwenden.
- **Persistenz liegt im Repo, nicht in Docker-Volumes**: `./ollama` ist nach `/root/.ollama` gemountet — Modell-Blobs (`ollama/models/blobs/`), Manifeste und die Chat-`history` landen also direkt im Arbeitsverzeichnis (Blobs sind zig GB groß). Deshalb ist `ollama/` per `.gitignore` ausgeschlossen; ebenso `files/*.gguf`. Modelle werden nicht versioniert, sondern per `ollama pull`/`ollama create` reproduziert. Open WebUI nutzt ein benanntes Volume `open-webui-data` — `docker compose down -v` löscht nur dieses, nicht die Ollama-Modelle.
- **GPU-Abhängigkeit**: Der Ollama-Service reserviert alle NVIDIA-GPUs (`deploy.resources.reservations.devices`). Das Setup setzt einen Host mit NVIDIA-Treiber + Container-Toolkit voraus (hier: WSL2). Ohne GPU müsste dieser Block entfernt werden.

## Modelfiles (`files/`)

`docker-compose.yml` mountet `./files` read-only nach `/files` (gleicher Name im Container, um Verwechslung mit Ollamas internem `ollama/models/`-Blob-Store zu vermeiden). Lege dort `.modelfile`-Definitionen ab, um Modelle anzupassen. Eine Modelfile leitet entweder von einem Ollama-Modell ab (`FROM nemotron-3-nano:30b`) oder von einer lokalen GGUF-Datei (`FROM /files/<datei>.gguf`) und überschreibt Parameter:
- `num_gpu` — Zahl der auf die GPU ausgelagerten Layer (Offloading; an den verfügbaren VRAM anpassen).
- `num_ctx` — Kontextfenster.

Nach Änderungen an einer Modelfile muss das Modell mit `ollama create <name> -f /files/<name>.modelfile` neu erzeugt werden, damit die Parameter greifen.

**Gotcha — `files/` gehört nach einem Recreate root**: Wenn das Verzeichnis fehlt, legt Docker es beim `up` als root an, und du kannst nicht hineinschreiben. Es leer neu anlegen (`rmdir files && mkdir files`) reicht, weil das Repo-Root dir gehört — kein `sudo` nötig. **Aber:** Ein laufender Container hält das Bind-Mount am alten Verzeichnis-Inode fest; nach `rmdir`/`mkdir` zeigt `/files` im Container ins Leere. Mit `docker compose up -d --force-recreate ollama` neu mounten (ein normales `up` erkennt keine Änderung und mountet **nicht** neu).

## Private/gated GGUF von HuggingFace laden

`ollama pull hf.co/<user>/<repo>:<quant>` funktioniert nur für **öffentliche** Repos. Sobald `HF_TOKEN` gesetzt ist (nötig für private/gated Repos), bricht der Pull mit `realm host "huggingface.co" does not match original host "hf.co"` ab — ein bekannter Ollama-Bug. Workaround: GGUF mit dem offiziellen `hf`-CLI (`pip install -U "huggingface_hub[cli]"`) ziehen und lokal registrieren.

```bash
set -a; . ~/.env.secrets; set +a          # HF_API_KEY laden
export HF_TOKEN="${HF_API_KEY}"           # `hf` liest HF_TOKEN, nicht HF_API_KEY

# 1. GGUF-Datei(en) des Repos prüfen (welcher Quant-Suffix existiert?)
hf models info <user>/<repo> --expand siblings \
  | python3 -c "import sys,json;[print(f['rfilename']) for f in json.load(sys.stdin)['siblings'] if f['rfilename'].endswith('.gguf')]"

# 2. Datei nach files/ laden (resumable bei Abbruch, mehrere zig GB)
hf download <user>/<repo> <datei>.gguf --local-dir files/

# 3. Modelfile mit FROM /files/<datei>.gguf anlegen, dann registrieren
docker compose exec ollama ollama create <name> -f /files/<name>.modelfile
```

Bei einem Multi-Part-Split (`...-00001-of-0000N.gguf`) alle Teile in dasselbe Verzeichnis laden (`--include "<basisname>-*.gguf"`) und im `FROM` nur die erste Teildatei referenzieren — Ollama findet die übrigen automatisch.

---
> Source: [soofi-project/soofi-model-hosting](https://github.com/soofi-project/soofi-model-hosting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
