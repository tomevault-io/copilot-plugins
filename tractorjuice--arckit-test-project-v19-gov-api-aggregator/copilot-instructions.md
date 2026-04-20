## arckit-test-project-v19-gov-api-aggregator

> **UK Government API Aggregator** — A platform that aggregates, indexes, and provides unified access to UK Government APIs from across departments and public sector organisations.

# CLAUDE.md

## Project Overview

**UK Government API Aggregator** — A platform that aggregates, indexes, and provides unified access to UK Government APIs from across departments and public sector organisations.

## Context

The UK Government maintains hundreds of APIs across dozens of departments (HMRC, NHS Digital, Companies House, Ordnance Survey, DVLA, Environment Agency, etc.), catalogued at https://www.api.gov.uk/. Currently, developers must discover, register for, and integrate with each department's API individually. This project creates a unified aggregation layer.

## Key Capabilities

- **API Discovery & Cataloguing**: Automatically discover and index UK Government APIs from api.gov.uk and department developer hubs
- **Unified Gateway**: Single entry point for accessing multiple government APIs with consistent authentication, rate limiting, and error handling
- **Developer Portal**: Self-service portal with documentation, sandbox environments, and API key management
- **Data Transformation**: Normalise responses across departments into consistent formats
- **Usage Analytics**: Track API consumption, latency, errors, and usage patterns
- **Compliance**: UK Government Technology Code of Practice (TCoP), GDS Service Standard, NCSC security guidance

## Architecture Commands

This project uses [ArcKit](https://github.com/tractorjuice/arc-kit) for architecture governance. Commands are available as slash commands in Claude Code, Gemini CLI, and Codex CLI.

```bash
# Start with architecture principles
/arckit.principles

# Define requirements
/arckit.requirements Government API Aggregator

# Discover data sources (APIs to aggregate)
/arckit.datascout Discover UK Government APIs for aggregation

# Full command list
# See docs/README.md or DEPENDENCY-MATRIX.md
```

## Build & Development

This is an architecture-first project. No application code exists yet — start with ArcKit commands to define the architecture before implementation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tractorjuice) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
