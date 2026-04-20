## curated-n8n-workflows

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a collection of 200+ ready-to-import n8n workflow templates organized by industry and use case. The repository contains JSON workflow files that can be directly imported into n8n automation platform.

## Architecture and Structure

### Organization
- **20+ categories** organized by industry/domain (AI_ML, Finance_Accounting, E_Commerce_Retail, etc.)
- Each category contains:
  - `*.json` files: n8n workflow definitions
  - `README.md`: Lists workflows with their tech stack dependencies

### Workflow Structure
Each JSON file is a complete n8n workflow containing:
- **Nodes**: Individual workflow steps (webhooks, AI models, databases, APIs)
- **Connections**: Data flow between nodes
- **Parameters**: Configuration for each node including credentials and settings
- **Position data**: Visual layout information for the n8n editor

### Common Tech Stack Components
- **Vector Stores**: Pinecone, Weaviate, Supabase Vector, Redis
- **LLM Providers**: OpenAI GPT-4(o), Anthropic Claude 3, Hugging Face
- **Embeddings**: OpenAI, Cohere, Hugging Face
- **Memory**: Zep Memory, Window Buffer
- **Integrations**: Slack, Google Sheets, various APIs

## Common Development Tasks

### Working with Workflows
- **Import**: JSON files are imported directly into n8n via Settings > Import Workflows
- **Testing**: Workflows are activated and tested within the n8n platform
- **Credentials**: Each node requiring external services needs credential configuration

### File Operations
- Workflow files are static JSON exports from n8n
- No build process or compilation required
- Direct file editing is possible but complex due to n8n's internal structure

### Quality Standards
- Workflows include error handling and guard-rails
- Documentation provided in category README files
- Many workflows implement complete RAG (Retrieval-Augmented Generation) stacks

## Contributing Guidelines

### Workflow Completion
- Some templates are marked as incomplete and need finishing
- Contributors should complete partial workflows following existing patterns
- Test workflows thoroughly before submission

### New Templates
- Follow existing category structure
- Update category README.md with tech stack information
- Ensure proper error handling and documentation
- Include credential placeholders for required services

## Important Notes

- This is a template repository, not a running application
- No package.json, build scripts, or test suites
- Workflows are executed within the n8n platform environment
- Each workflow is self-contained with its own dependencies defined in the JSON structure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MaTriXy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
