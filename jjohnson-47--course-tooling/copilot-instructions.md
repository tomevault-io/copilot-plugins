## course-tooling

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **Course-Tooling Hub** - a centralized Agentic Course Deployment System that provides reusable GitHub Actions workflows and templates for deploying educational course websites. The system implements a hub-and-spoke architecture where this repository serves as the hub containing shared deployment infrastructure.

## Architecture

### Hub-and-Spoke Topology
- **Hub**: `course-tooling` (this repo) - contains reusable workflows and templates
- **Spokes**: Course repositories following the pattern:
  - `<course>-src` (private) - source Markdown content
  - `<course>-site` (public) - generated GitHub Pages sites

### Environment Integration
The system integrates with an Agentic Development Environment (ADE):
- Source repositories: `$AGENTIC_ENV_ROOT/courses/`
- Runtime state: `$AGENTIC_STATE_ROOT/cds/`
- Required environment variables: `CDS_ENV_ROOT` and `CDS_STATE_ROOT`

## Key Commands

### Architecture Validation
```bash
./scripts/validate-architecture.sh
```
This script validates:
- `cds_agent` daemon is running
- MCP heartbeat connectivity
- No literal paths outside designated environment variables
- Must pass for all merges (CI enforcement)

### Environment Bootstrap
```bash
source ./scripts/bootstrap-env.sh
```
One-time setup for fresh machine initialization.

## File Structure

- `A0.md` - Immutable Baseline Architectural Constitution
- `bootstrap-guide.md` - Setup instructions for fresh environments
- `.github/workflows/reusable-deploy-workflow.yml` - Reusable deployment workflow for course repositories
- `templates/` - Pandoc HTML templates and CSS for course site generation
- `scripts/` - Validation and bootstrap scripts

## Development Notes

### Workflow Usage Pattern
Course repositories invoke the reusable workflow:
```yaml
uses: <org>/course-tooling/.github/workflows/reusable-deploy-workflow.yml@baseline-v1
with:
  PUBLIC_REPO: <org>/<course>-site
secrets: inherit
```

### Security Model
- Fine-grained PATs stored in macOS Keychain
- Secrets separated between `cds_agent` and individual course repos
- No secrets stored in this central repository

### MCP Integration
The system includes a `cds_agent` daemon that:
- Sends heartbeats every 30 seconds to MCP
- Polls GitHub API every 5 minutes for deployment status
- Processes deployment artifacts and detects anomalies

### Architecture Immutability
The A0 baseline is immutable. Changes to core architecture require a new baseline version with appropriate Git tagging (e.g., `baseline-v2`).

---
> Source: [jjohnson-47/course-tooling](https://github.com/jjohnson-47/course-tooling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
