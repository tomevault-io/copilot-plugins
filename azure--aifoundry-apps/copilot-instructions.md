## aifoundry-apps

> An agentic application that empowers solution engineers to customize multi-agent patterns and leverage the Azure AI Foundry Agent Service Catalog of Agent templates using SWE (Software Engineering) Agents.

# AIFoundry Apps - AI Agent Platform

An agentic application that empowers solution engineers to customize multi-agent patterns and leverage the Azure AI Foundry Agent Service Catalog of Agent templates using SWE (Software Engineering) Agents.

## Project Overview

- **Type**: Full-stack AI agent platform
- **Purpose**: Template-based agent deployment and multi-agent pattern customization
- **Target Users**: Solution engineers, developers working with AI agents
- **Deployment**: Azure Container Apps with marketplace integrations

## Architecture

- **Backend**: FastAPI with Python, integrated with Azure AI Foundry services
- **Frontend**: Vite + React + TypeScript + Tailwind CSS  
- **Agent Layer**: MCP-based integration with multiple AI agents (Copilot, Devin, Codex)
- **Infrastructure**: Azure Container Apps, GitHub Actions, Docker

## Core Technologies & Integrations

### AI Agents & Tools
- **GitHub Copilot Agent**: Integrated via MCP (Model Context Protocol) for intelligent code suggestions
- **Azure AI Foundry Model**: Powered by Codex for advanced code generation and analysis
- **Cognition Devin**: Deployed via MCP on Azure Marketplace for autonomous software engineering
- **Replit Integration**: Available through Azure Marketplace for cloud-based development

### Development Infrastructure
- **GitHub Actions Runner**: Automated CI/CD pipelines and testing
- **MCP (Model Context Protocol)**: Seamless integration between different AI agents
- **Azure Marketplace**: Deployment and distribution platform for agent services

## Key Features

- **Agent Template Gallery**: Browse and discover pre-built agent templates from Azure AI Foundry
- **Multi-Agent Pattern Customization**: Tailor agent behaviors and workflows to specific use cases
- **Integrated Development Environment**: Test and iterate on agent solutions in real-time
- **SWE Agent Integration**: Leverage Software Engineering Agents for automated code generation
- **Modern UI/UX**: Clean, intuitive interface inspired by Azure AI Labs design language

## Development Setup

### Prerequisites
- Azure AI Foundry access
- GitHub Copilot subscription
- Azure Marketplace account for agent services
- Node.js and Python development environment

### Quick Start
```bash
# Backend
cd src/backend
uv venv && source .venv/bin/activate
uv pip install -e .
uv run fastapi dev app/main.py

# Frontend  
cd src/frontend
pnpm install
pnpm run dev
```

## Deployment Options

- **Local Development**: Docker Compose setup
- **Azure Container Apps**: Production deployment via Azure Developer CLI
- **GitHub Actions**: Automated CI/CD pipelines

## Configuration

- **Use MCP Tools**: Yes - Core integration pattern
- **Use A2A**: Yes - Agent-to-Agent communication enabled
- **Authentication**: GitHub OAuth integration
- **API Integration**: Azure AI Foundry, GitHub API, MCP servers

## Repository Structure

```
.
├── src/
│   ├── backend/           # FastAPI backend with AI agent integrations
│   └── frontend/          # React frontend for agent template management
├── deployment/            # Deployment scripts and configs
├── infra/                # Azure infrastructure templates
└── azure.yaml           # Azure Developer CLI config
```

## Live Demo

🚀 **Production**: [aifoundry.app](https://aifoundry.app)

---
> Source: [Azure/aifoundry-apps](https://github.com/Azure/aifoundry-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
