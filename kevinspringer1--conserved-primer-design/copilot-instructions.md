## conserved-primer-design

> This repository is a VirusPrimerPro Agent API skill package.

# Copilot Instructions

This repository is a VirusPrimerPro Agent API skill package.

When generating code or commands:

- Use `agentskill/scripts/vpp_agent_api.py` for API calls instead of handwritten upload/polling code.
- Keep API keys in `VPP_AGENT_API_KEY`; never commit secrets.
- Use asynchronous polling for long-running VirusPrimerPro jobs.
- Use `agentskill/scripts/ncbi_mafft.py` for NCBI + MAFFT alignment preparation.
- Use `agentskill/scripts/curate_references.py` before creating genotype or PhyloGuide group files.
- Use `agentskill/scripts/primer_panel_qc.py` before reporting primer or tNGS panel designs as ready.
- Prefer English documentation and command examples.

---
> Source: [KevinSpringer1/conserved-primer-design](https://github.com/KevinSpringer1/conserved-primer-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
