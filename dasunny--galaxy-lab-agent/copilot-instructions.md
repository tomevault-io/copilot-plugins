## galaxy-lab-agent

> AI agent for Dr. Sun's Computational Genomics Lab at ODU — Galaxy workflow discovery, execution, and monitoring


# ODU Computational Genomics Lab Agent

## Context
This agent connects to the lab's Galaxy instance running on ODU's HPC cluster at
jsun-compute-0.cs.odu.edu. It supports any bioinformatics workflow the lab runs —
genomics, epigenomics, transcriptomics, or otherwise.

## What This Agent Can Do
1. **Discovery** — answer questions about available tools and workflows from local catalog
2. **Execution** — invoke Galaxy workflows via BioBlend
3. **Monitoring** — check job status and history
4. **Reporting** — generate reproducible output bundles for every run

## What Requires a Live API Call
- Invoking a workflow
- Checking job or dataset status
- Fetching history details
- Anything time-sensitive or that may have changed since last sync

## What Does NOT Require a Live API Call
- Tool discovery and search (use galaxy_catalog.json)
- Workflow structure questions (use workflow_catalog.json)
- Tool parameter details (use galaxy_skills/)
- General methodology questions

## Lab Infrastructure
- Galaxy instance: jsun-compute-0.cs.odu.edu
- HPC cluster: ODU research computing
- Galaxy version: 24.2

## Principles
- Never guess tool IDs — always use the catalog
- Always generate a reproducibility bundle for executions
- Prefer local catalog lookups over live API calls where possible

## Roadmap (Future Capabilities)

### v0.1 — Discovery + Monitoring (current)
- Natural language tool and workflow discovery from local catalog
- Check job status and histories
- Answer methodology questions without hitting the API

### v0.2 — Workflow Execution
- Invoke existing Galaxy workflows via BioBlend
- Poll for completion and report status
- Generate reproducibility bundles for every run

### v0.3 — Individual Tool Invocation
- Run single Galaxy tools on demand
- Handle tool-specific parameter schemas
- Chain tool outputs into histories

### v0.4 — Workflow Assembly from Templates
- Suggest multi-tool pipelines for a described task
- Assemble and import new .ga workflow files
- Build from curated templates per analysis type

### v1.0 — Dynamic Workflow Construction
- Agent reasons about biological question
- Designs pipeline from scratch
- Constructs, imports, and executes novel workflows

---
> Source: [dasunny/galaxy-lab-agent](https://github.com/dasunny/galaxy-lab-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
