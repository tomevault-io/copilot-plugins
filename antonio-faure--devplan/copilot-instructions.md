## devplan

> - Langue : français pour la communication, anglais pour le code

# DevPlan — Agent Instructions

## Conventions générales

- Langue : français pour la communication, anglais pour le code
- Méthode : Code, Documente et Commit pour chaque changement
- Préfixe d'exception : `raise ... from e` pour le chaînage
- Type hints : absents (le code restant ne les utilise pas)
- Commentaires : ne pas en ajouter

## Commandes

Toutes les commandes passent par le venv : `.venv/bin/`.

| Commande | Description |
|----------|-------------|
| `.venv/bin/python -m pytest tests/ -v` | Lancer les tests unitaires |
| `.venv/bin/pip install -e voxtral-subtitles` | Réinstaller le package en mode dev |
| `.venv/bin/voxtral-subtitles --help` | Afficher l'aide du CLI |
| `git status` / `git diff` | Vérifier l'état avant commit |
| `git push origin master` | Pusher sur GitHub |

## Structure du projet

```
DevPlan/
├── README.md                  # Roadmap globale
├── AGENTS.md                  # Ce fichier
├── SponsorDetection.md        # Documentation pipeline détection sponsors
├── Plan pour le RAG sur vidéos YT.md
├── .venv/                     # Environnement virtuel Python
└── voxtral-subtitles/         # Étape 1 ✅
    ├── src/voxtral_subtitles/
    │   ├── __init__.py
    │   ├── __main__.py
    │   ├── cli.py             # CLI (argparse, 3 commandes)
    │   └── utils.py           # Logique métier
    ├── tests/
    │   ├── __init__.py
    │   └── test_utils.py      # 37 tests unitaires
    ├── pyproject.toml
    ├── requirements.txt
    └── README.md
```

## Conventions voxtral-subtitles

- CLI via argparse, 3 commandes : `transcribe`, `vo-subtitles`, `multilingual-subtitles`
- Toute commande accepte `input_file_path` (positionnel optionnel) et `--url` (mutuellement exclusifs via erreur)
- Flags communs : `--keep-temp`, `--verbose`, `--diarize`
- Valeurs par défaut : Verdana 18px, highlight `#F5A623`, incoming `#999999`, alignment `Bottom Center`
- Client Mistral lazy (via `@cache` + `get_client()`) pour ne pas crasher sur `--help` sans API key
- Variables d'environnement : `MISTRAL_API_KEY` ou `mistral_api_key`
- `cleanup_files()` appelé en début de commande (et en fin si `--keep-temp` absent)
- Output dans `videos/` quand on utilise `--url`

## Dépendances principales

- mistralai >= 2.4.0
- pysubs2 >= 1.7.0
- pydantic >= 2.0.0
- pytest >= 8.0.0 (dev)

## Outils système requis

- **ffmpeg** avec `libass` : `~/bin/ffmpeg` (static build GitHub, à jour). Détecté automatiquement par `_find_ffmpeg()`.
- **yt-dlp** : `~/bin/yt-dlp` (static build GitHub, à jour). Détecté par `_find_ytdlp()`, fallback sur `.venv/bin/yt-dlp`.

## API Keys

- Mistral : `MISTRAL_API_KEY` (variable d'environnement, ne jamais commiter de clé)
- GitHub : token classique, gh CLI dans `C:\Program Files\GitHub CLI\gh.exe`

---
> Source: [Antonio-Faure/DevPlan](https://github.com/Antonio-Faure/DevPlan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
