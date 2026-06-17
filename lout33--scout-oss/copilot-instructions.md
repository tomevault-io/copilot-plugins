## scout-oss

> Run a local web research agent and mission-driven intelligence scanner that searches multiple sources and writes markdown briefs. Best for trend monitoring, opportunity scouting, builder discovery, and technical signal scanning.


# Scout

Mission-driven local web research agent backed by the files in this directory.

## What it does

- plans a scan from a mission
- calls source adapters to gather signal
- follows promising leads
- writes a markdown brief to `inbox/`

Best for:

- technical trend monitoring
- open-source discovery
- opportunity scouting
- builder and ecosystem research

## Working adapters

- Hacker News
- GitHub
- Jina Reader

## Setup

```bash
pip install -r requirements.txt
cp .env.example .env
```

Then configure one of:

- Codex auth: `codex login --device-auth`
- `ANTHROPIC_API_KEY`

Optional:

- `JINA_API_KEY`
- `GITHUB_TOKEN`

## Direct commands

```bash
python agent.py --mission "Find 3 interesting technical signals from today" --tokens 5000
./quick.sh --mission "Find 2 interesting GitHub repos from this week"
./scout.sh --tokens 20000 --mission "Find open-source agent tooling worth tracking"
./daily.sh --tokens 10000
```

## Recommended smoke test

```bash
python agent.py --mission "Use hn_front, github_trending, and jina_read to find 2 interesting technical signals from today and write a brief." --tokens 2500
```

If the setup works, Scout writes a markdown brief to `inbox/`.

---
> Source: [lout33/scout-oss](https://github.com/lout33/scout-oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
