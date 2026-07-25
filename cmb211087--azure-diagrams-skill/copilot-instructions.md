## azure-diagrams-skill

> Generates Azure architecture diagrams (800+ Azure service icons), plus business-process

# AGENTS.md - Azure Architecture Diagrams

This repository is an **agent skill** for generating professional technical diagrams as
code. It works with any AI coding tool (Claude Code, GitHub Copilot, Cursor, Windsurf, and
any tool that reads `AGENTS.md` or Agent Skills). The full instructions live in
[`skills/azure-diagrams/SKILL.md`](skills/azure-diagrams/SKILL.md); this file is the
portable summary for tools that do not load `SKILL.md` automatically.

## What it does

Generates Azure architecture diagrams (800+ Azure service icons), plus business-process
flows, ERDs, timelines/Gantt, UI wireframes, sequence diagrams and network topology, using
the Python [`diagrams`](https://diagrams.mingrammer.com/) library and Graphviz. It can also
build diagrams from Bicep, Terraform and ARM templates.

## Prerequisites (cross-platform)

1. **Graphviz** (system package, provides the `dot` engine), then restart the terminal:
   - Windows: `winget install -e --id Graphviz.Graphviz` (or `choco install graphviz`)
   - macOS: `brew install graphviz`
   - Linux: `sudo apt-get install graphviz`
2. **Python 3.9+** with the library: `pip install "diagrams>=0.25.1"`

See [`SETUP.md`](SETUP.md) for a full, step-by-step setup with troubleshooting.

## How to generate a diagram

Write the image to a `diagrams/` folder in the working directory (portable everywhere); do
not leave a throwaway `.py` script behind.

**macOS / Linux (and Claude Code, Cursor):**
```bash
mkdir -p diagrams
python3 << 'EOF'
from diagrams import Diagram
from diagrams.azure.compute import AKS
from diagrams.azure.database import CosmosDb
with Diagram("My Architecture", filename="diagrams/architecture", show=False, outformat="png"):
    AKS("aks-prod") >> CosmosDb("cosmos-prod")
EOF
```

**Windows (PowerShell, e.g. GitHub Copilot):**
```powershell
New-Item -ItemType Directory -Force diagrams > $null
@'
from diagrams import Diagram
from diagrams.azure.compute import AKS
from diagrams.azure.database import CosmosDb
with Diagram("My Architecture", filename="diagrams/architecture", show=False, outformat="png"):
    AKS("aks-prod") >> CosmosDb("cosmos-prod")
'@ | Set-Content _diagram.py; python _diagram.py; Remove-Item _diagram.py
```

PNG is the default output (portable and shareable). See the reference files under
`skills/azure-diagrams/references/` for patterns, the full component catalogue, and
diagram-type guides.

## Conventions for agents

- Prefer PNG output; only use SVG when the user asks and `cairosvg` is installed.
- Use `python skills/azure-diagrams/scripts/list_azure_nodes.py` to look up real icon
  class names instead of guessing.
- Keep node labels short; use `Cluster` for logical grouping; set `labelloc="t"`.
- Verify the environment with `python skills/azure-diagrams/scripts/verify_installation.py`.

---
> Source: [cmb211087/azure-diagrams-skill](https://github.com/cmb211087/azure-diagrams-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
