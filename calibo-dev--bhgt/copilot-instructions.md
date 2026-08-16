## bhgt

> This project implements a multi-agent system using CrewAI framework to research and report on various topics. The system consists of two specialized agents working in a sequential process to gather information and generate comprehensive reports.

# Agents Documentation

## Overview

This project implements a multi-agent system using CrewAI framework to research and report on various topics. The system consists of two specialized agents working in a sequential process to gather information and generate comprehensive reports.

## Architecture

The agent system is built on CrewAI 1.8.1 and uses:
- **LLM Provider Selection**: Controlled by `PROVIDER` environment variable
  - `PROVIDER=OPENAI`: Uses OpenAI GPT-4.1-mini (requires API key secret retrieval)
  - `PROVIDER=ANTHROPICAI`: Uses Anthropic models (requires API key secret retrieval)
  - `PROVIDER=AZUREAI`: Uses Azure models (requires API key secret retrieval)
  - `PROVIDER=GEMINIAI`: Uses Gemini models (requires API key secret retrieval)
  - `PROVIDER=OLLAMA` (default): Uses Ollama with llama3.2 model
- **Process Type**: Sequential
- **Framework**: CrewAI with FastAPI integration
- **Python Version**: 3.10 - 3.13

## Agents

### 1. Researcher Agent

**Role**: `{topic} Senior Data Researcher`

**Goal**: Uncover cutting-edge developments in the specified topic

**Backstory**: 
A seasoned researcher with a knack for uncovering the latest developments in any given topic. Known for the ability to find the most relevant information and present it in a clear and concise manner.

**Configuration**:
- LLM: `ollama/llama3.2`
- Temperature: 0.7
- Verbose: True

**Primary Task**: Research Task
- Conducts thorough research about the specified topic
- Finds interesting and relevant information for the current year
- Outputs: A list with 10 bullet points of the most relevant information

### 2. Reporting Analyst Agent

**Role**: `{topic} Reporting Analyst`

**Goal**: Create detailed reports based on data analysis and research findings

**Backstory**: 
A meticulous analyst with a keen eye for detail. Known for the ability to turn complex data into clear and concise reports, making it easy for others to understand and act on the information provided.

**Configuration**:
- LLM: `ollama/llama3.2`
- Temperature: 0.7
- Verbose: True

**Primary Task**: Reporting Task
- Reviews the research context and expands each topic into a full section
- Creates detailed reports containing all relevant information
- Outputs: A fully-fledged markdown report with main topics and detailed sections
- Output File: `report.md`

## Workflow

The agents work in a sequential process:

```
1. Researcher Agent
   └─> Conducts research on the topic
       └─> Generates 10 bullet points of key findings
           └─> Passes context to next agent

2. Reporting Analyst Agent
   └─> Reviews research findings
       └─> Expands each point into detailed sections
           └─> Generates final markdown report (report.md)
```

## Configuration Files

### agents.yaml
Located at: `src/latest_ai_development/config/agents.yaml`

Defines agent roles, goals, backstories, and LLM configurations.

### tasks.yaml
Located at: `src/latest_ai_development/config/tasks.yaml`

Defines task descriptions, expected outputs, and agent assignments.

## Usage

### Running the Crew

The system can be executed in multiple ways:

#### 1. Direct Execution
```python
from latest_ai_development.crew import LatestAiDevelopment
from datetime import datetime

inputs = {
    'topic': 'Your Topic Here',
    'current_year': str(datetime.now().year)
}

LatestAiDevelopment().crew().kickoff(inputs=inputs)
```

#### 2. FastAPI Endpoint
```bash
POST /ask
{
    "topic": "Your Topic Here"
}
```

#### 3. Command Line
```bash
# Run the crew
latest_ai_development

# Train the crew
train <n_iterations> <filename>

# Replay a specific task
replay <task_id>

# Test the crew
test <n_iterations> <eval_llm>

# Run with trigger payload
run_with_trigger '<json_payload>'
```

## Environment Variables

- `PROVIDER`: LLM provider selection (OPENAI, ANTHROPICAI, AZURE, GEMINIAI, or OLLAMA, default: OLLAMA)
- `CLOUD_PROVIDER`: Secret backend selection (`AWS` or `AZURE`)
- `API_KEY_SECRET`: Preferred secret name used to retrieve the provider API key
- `OPENAI_API_KEY_SECRET`: Backward-compatible fallback secret name for OpenAI when `API_KEY_SECRET` is not set
- `ANTHROPICAI_API_KEY_SECRET`: Backward-compatible fallback secret name for Anthropic when `API_KEY_SECRET` is not set
- `AZUREAI_OPENAI_API_KEY_SECRET`: Backward-compatible fallback secret name for Azure OpenAI when `API_KEY_SECRET` is not set
- `GEMINIAI_API_KEY_SECRET`: Backward-compatible fallback secret name for Gemini when `API_KEY_SECRET` is not set
- `OLLAMA_BASE_URL`: Base URL for Ollama API (used when PROVIDER=OLLAMA)
- `TRACING_BACKEND`: Tracing backend selector (`LANGSMITH`, `CREWAI`, or `NONE`). When unset or blank, the app falls back to `CREWAI`.
- `LANGSMITH_ENDPOINT`: LangSmith API endpoint (for example `https://api.smith.langchain.com`)
- `LANGSMITH_PROJECT`: LangSmith project name for trace grouping
- `LANGSMITH_API_KEY`: Direct LangSmith API key value
- `LANGSMITH_API_KEY_SECRET`: Preferred secret name used to resolve the LangSmith API key through the configured secret manager
- `PORT`: Server port (default: 8080)
- `AWS_REGION`: AWS region for Secrets Manager (default: us-east-1)
- `AZURE_KEY_VAULT_URL`: Azure Key Vault URL (required when `CLOUD_PROVIDER=AZURE`)

## Integration Features

### FastAPI Server
- Health check endpoint: `GET /health`
- Research endpoint: `POST /ask`
- Root path: `/testing`

### LangSmith Tracing
- Tracing initialization happens in `src/latest_ai_development/main.py` during application startup.
- When `TRACING_BACKEND=LANGSMITH`, the app configures LangSmith and instruments OpenAI through OpenTelemetry.
- When LangSmith tracing is active, `main.py` disables CrewAI anonymous telemetry for that process to avoid overlapping telemetry initialization.
- Use `LANGSMITH_API_KEY_SECRET` when the LangSmith API key is stored in AWS Secrets Manager or Azure Key Vault.
- Use `LANGSMITH_API_KEY` only when supplying the LangSmith API key value directly as an environment variable.

### CrewAI Tracing
- When `TRACING_BACKEND=CREWAI`, the app enables CrewAI native tracing through the `Crew(...)` configuration in `src/latest_ai_development/crew.py`.
- When `TRACING_BACKEND` is unset or blank, the app defaults to CrewAI native tracing.
- When `TRACING_BACKEND=LANGSMITH`, CrewAI native tracing stays disabled to avoid overlapping tracing systems.
- CrewAI's own terminal tracing banner still reflects CrewAI native tracing only. It does not report LangSmith status.

### Secret Management
The system includes a `SecretsManager` class for secure credential management:
- Retrieves secrets from AWS Secrets Manager or Azure Key Vault
- Selects the backend using the `CLOUD_PROVIDER` environment variable
- Supports both full secret retrieval and specific key extraction
- Uses `AWS_REGION` for AWS and `AZURE_KEY_VAULT_URL` for Azure

### Custom Tools
The framework supports custom tool development through the `BaseTool` class. Example template available at `src/latest_ai_development/tools/custom_tool.py`.

## Output

The final output is a markdown report (`report.md`) containing:
- Main topics identified during research
- Detailed sections for each topic
- All relevant information and findings
- Formatted without code blocks for clean presentation

## Deployment

The project includes:
- Docker support (Dockerfile, .dockerignore)
- Kubernetes Helm charts (helm_chart/)
- Jenkins CI/CD pipelines (Jenkinsfile, Jenkinsfile.ci, Jenkinsfile.deploy)

## Dependencies

Key dependencies:
- `crewai[tools]==1.8.1` - Core framework
- `fastapi>=0.128.0` - API server
- `uvicorn>=0.40.0` - ASGI server
- `litellm>=1.75.3` - LLM integration
- `boto3>=1.42.47` - AWS integration
- `azure-identity>=1.25.1` - Azure authentication
- `azure-keyvault-secrets>=4.10.0` - Azure Key Vault integration
- `anthropic>=0.109.1` - Anthropic integration
- `apscheduler>=3.11.2` - Task scheduling

## Extending the System

### Adding New Agents
1. Define agent configuration in `agents.yaml`
2. Create agent method in `crew.py` with `@agent` decorator
3. Assign tasks to the new agent

### Adding New Tasks
1. Define task configuration in `tasks.yaml`
2. Create task method in `crew.py` with `@task` decorator
3. Specify agent assignment and expected output

### Creating Custom Tools
1. Extend `BaseTool` class
2. Define input schema using Pydantic
3. Implement `_run()` method
4. Add tool to agent configuration

## Restrictions and Guidelines

### Environment Variables
- **PORT**: MUST be read from environment variable. Do not hardcode port values.
- **CONTEXT**: Context path is MANDATORY for running the agent. Must be set as an environment variable.

### Dockerfile
- The Dockerfile start command MUST NOT be modified
- Any changes made to the Dockerfile MUST adhere to the existing start command
- Maintain compatibility with the current container startup process

### Protected Files
The following files and directories are PROTECTED and should NOT be modified:
- `Jenkinsfile`
- `Jenkinsfile.ci`
- `Jenkinsfile.deploy`
- `helm_chart/` (entire directory and all contents)

These files are managed by the DevOps team and any changes could break the CI/CD pipeline or deployment process.

## Notes

- The system uses a sequential process by default
- Hierarchical process is available as an alternative
- All agents share the same LLM configuration
- Knowledge sources can be added to enhance agent capabilities
- The system supports training, testing, and replay functionality for iterative improvement

---
> Source: [calibo-dev/bhgt](https://github.com/calibo-dev/bhgt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
