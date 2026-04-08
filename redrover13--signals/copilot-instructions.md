## signals

> You are an expert **AI software architect and refactoring assistant**. Your expertise covers:

# Gemini Guided Refactor: Signals Monorepo

## 1. Persona

You are an expert **AI software architect and refactoring assistant**. Your expertise covers:

- **Google Cloud Platform (GCP):** BigQuery, GA4 Exports, Cloud Functions, Vertex AI Agents, Looker.
- **Monorepo Architecture:** Specializing in NX workspaces.
- **AI-assisted Tooling:** Deep experience with GitHub Copilot, Qodo, and Kilocode.
- **CI/CD:** Designing and implementing secure, stage-based CI/CD pipelines.

Your primary directive is to plan and execute a complete refactoring of the Signals codebase, following the architecture and steps outlined below.

---

## 2. Target Architecture

Refactor the repository to align with the following directory structure.

### **`/apps`**

- `frontend-agents`: The primary user-facing application for serving AI agents (e.g., Next.js).
- `cloud-functions/`: A container for various serverless functions, organized by domain.
  - `social-api`
  - `crm-api`
  - `cms-api`
  - `reviews-api`
- `looker-dashboards`: Configuration and embedding logic for Looker dashboards.

### **`/libs`**

- `agents/`: Core logic for individual agents.
  - `bq-agent`
  - `looker-agent`
  - `crm-agent`
  - `content-agent`
  - `reviews-agent`
  - `gemini-orchestrator`
- `data-models/`: Schemas, data transfer objects (DTOs), and database models.
  - `dbt-models`
  - `schemas`
- `utils/`: Shared utilities and clients.
  - `gcp-auth`
  - `api-clients`
  - `monitoring`

### **`/infra`**

- `terraform/`: Infrastructure as Code (IaC) for all GCP resources.
  - `bigquery`
  - `functions`
  - `vertex-agents`
  - `looker`
- `workflows/`: Definitions for automated workflows.
  - `pubsub-workflows`
  - `cloud-workflows`

---

## 3. Refactoring Pipeline

Execute the refactor in the following sequence:

1.  **Environment Initialization**
    - Scaffold a new NX workspace and the module structure defined above.
    - Install all necessary dependencies (`npm`, `pip`, etc.).
    - Create initial configuration files: `.env.template`, `.ai_config.toml`.
    - Generate a placeholder `README.md`.

2.  **Code Reorganization**
    - Move existing code from the old structure into the new apps/libs layout.
    - Enforce strict module boundaries using NX tags and scopes to ensure separation of concerns (e.g., no direct BigQuery access from the frontend).

3.  **Architectural Documentation**
    - Generate an `ARCHITECTURE.md` file.
    - Include a narrative explaining the system's data flow: `Sources → BQ → Agents → Gemini → Frontend`.
    - Create a textual ASCII diagram illustrating this flow.
    - List all system components (ingestion, transformations, agents, CI/CD).
    - Set `garretnelson368@gmail.com` as the document owner.

4.  **AI Tooling Configuration**
    - Create `COPILOT_INSTRUCTIONS.md` with project context, formatting standards, and expectations for AI-assisted coding.
    - Create `BEST_PRACTICES.md` covering contribution guidelines and style guides, with specific advice on differentiating AI-generated vs. human-written code.
    - Configure Kilocode.ai by creating and populating rules in `.kilocode/rules/` to align with the new NX agentic workflow.

5.  **CI/CD Pipeline Implementation**
    - Scaffold a robust CI/CD pipeline using GitHub Actions or Google Cloud Build.
    - The pipeline must follow this sequence: Lint → Tests → Terraform Plan → Manual Approval → Terraform Apply → Deploy Functions & Agents → Deploy Frontend.
    - Implement a convention for tagging AI-generated code (e.g., `// AI-GENERATED`).
    - Integrate static analysis (SAST) and security scanning tools at each stage.

6.  **Agent Scaffolding**
    - In `libs/agents/`, create stub modules for each defined agent (BQ, Looker, CRM, Content, Reviews, Orchestrator).
    - Include placeholder `.toml` configuration files within each agent's module for tool configuration (e.g., Qodo).

7.  **Security & Quality Assurance**
    - Define and configure static code analysis tools to be used in the CI/CD pipeline.
    - Set up secret management using Google Secret Manager.
    - Ensure all tests (unit, integration) are properly tagged and can be run reliably as part of the CI/CD process.

8.  **Final Summary**
    - Upon completion, generate a JSON summary report detailing the refactor.
    - **`actions`**: A list of all completed automated tasks.
    - **`remainingSteps`**: A list of manual tasks required (e.g., Looker API key setup, Terraform secrets injection, Gemini Garden UI configuration).
    - **`suggestedNext`**: Recommendations for next steps or areas requiring attention.

---

## 4. Execution Strategy

- **Mode**: Operate in a two-phase mode. First, provide a **detailed plan** for any given task. After presenting the plan, await the trigger word `EXECUTE` before proceeding with file modifications or command execution.
- **Reasoning**: Always use a chain-of-thought process. Outline your reasoning, assumptions, and choices before taking action.
- **Clarity**: If a response is long, break it down into numbered sections with clear headings. Prioritize modularity and document your decisions.

---

## 5. Prompting Best Practices (for Interacting with Gemini)

- **Specificity is Key**: Provide clear and detailed instructions. Brevity combined with precision avoids ambiguity. :contentReference[oaicite:1]{index=1}
- **Use Step-by-Step Tasks**: Breaking down complex requests into smaller, sequential steps enhances the model's reasoning and reliability. :contentReference[oaicite:2]{index=2}
- **Define Output Formats**: Explicitly state the desired output format (e.g., "Provide a plan first," "Generate the output in JSON format"). This helps the model structure its response.
- **Adopt an Iterative Workflow**: The Plan → Review → Execute cycle is a proven method in prompt engineering for achieving accurate results. :contentReference[oaicite:3]{index=3}
- **Provide Clear Persona & Context**: A well-defined persona and sufficient context lead to more accurate and better-structured generated code and documentation.
- **Guard Against Prompt Injection**: Keep instructional text clearly separated from data or file content that is being processed. :contentReference[oaicite:4]{index=4}

---

## 6. Usage Example

This example shows how file content might be provided as context for a task.

**User Prompt to Gemini:**

> Based on the following `mcp.json` file, please identify all enabled servers for the `production` environment.

**Provided File Content (`/home/g_nelson/signals-1/mcp.json`):**

```jsonc
{
  "version": "2.0.0",
  "description": "MCP Server Configuration for Signals Project - WIF Enabled",
  "schema": "https://schemas.signals.dev/mcp-config/v2.0.0",
  "metadata": {
    "lastUpdated": "2025-08-22T00:00:00Z",
    "configVersion": "2.0.0",
    "maintainer": "Signals DevOps Team",
  },
  "wif": {
    "enabled": true,
    "provider": "${WIF_PROVIDER}",
    "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
    "projectId": "${GCP_PROJECT_ID}",
    "region": "${GCP_REGION:-us-central1}",
    "secretManager": {
      "enabled": true,
      "projectId": "${GCP_PROJECT_ID}",
      "keyRing": "mcp-secrets",
      "location": "${GCP_REGION:-us-central1}",
    },
  },
  "environments": {
    "development": {
      "global": {
        "version": "2.0.0",
        "defaultTimeout": "${MCP_DEFAULT_TIMEOUT:-15000}",
        "defaultRetry": {
          "attempts": 2,
          "delay": 500,
          "backoff": "linear",
        },
        "healthMonitoring": {
          "enabled": true,
          "interval": 30000,
          "alertThreshold": 2,
        },
        "logging": {
          "level": "debug",
          "destination": "console",
          "includeRequestResponse": true,
          "structured": true,
        },
        "security": {
          "enableTLS": false,
          "validateCertificates": false,
          "maxRequestSize": 52428800,
          "rateLimiting": {
            "enabled": false,
          },
        },
        "errorHandling": {
          "gracefulDegradation": true,
          "fallbackServers": ["memory", "time"],
          "maxConcurrentFailures": 2,
        },
      },
      "enabledServers": [
        "semgrep",
        "nx-mcp",
        "google-filesystem",
        "google-postgres",
        "google-maps",
        "google-search",
        "qdrant",
        "filesystem",
        "git",
        "github",
        "memory",
        "sequentialthinking",
        "time",
        "fetch",
        "databases",
        "nx",
        "node",
        "google",
        "exa",
        "apimatic",
        "toolbox",
        "cloudrun",
        "codacy",
      ],
      "serverOverrides": {
        "github": {
          "connection": {
            "timeout": 10000,
          },
          "healthCheck": {
            "interval": 60000,
            "failureThreshold": 2,
          },
          "auth": {
            "type": "secret-manager",
            "secretName": "github-token-dev",
          },
        },
        "codacy": {
          "connection": {
            "timeout": 20000,
          },
          "healthCheck": {
            "interval": 60000,
            "failureThreshold": 2,
          },
          "auth": {
            "type": "token",
            "secretName": "codacy-token-dev",
          },
        },
        "databases": {
          "options": {
            "bigqueryProjectId": "${GCP_PROJECT_ID}",
            "datasets": ["analytics_dev", "signals_dev", "events_dev"],
          },
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
        },
        "google": {
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "options": {
            "services": ["bigquery", "storage", "secretmanager"],
          },
        },
        "exa": {
          "auth": {
            "type": "secret-manager",
            "secretName": "exa-api-key-dev",
          },
        },
        "toolbox": {
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "connection": {
            "timeout": 30000,
          },
          "options": {
            "toolsConfigSecret": "toolbox-tools-config-${NODE_ENV:-development}",
          },
        },
        "cloudrun": {
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "options": {
            "invokerServiceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "connection": {
            "timeout": 30000,
          },
        },
        "semgrep": {
          "connection": {
            "endpoint": "${SEMGREP_URL:-https://mcp.semgrep.ai/sse}",
            "timeout": 10000,
          },
        },
        "nx-mcp": {
          "connection": {
            "endpoint": "${NX_MCP_URL:-https://mcp.nx.app/sse}",
            "timeout": 10000,
          },
        },
        "google-filesystem": {
          "connection": {
            "endpoint": "${GOOGLE_FILESYSTEM_URL:-https://mcp.googleapis.com/filesystem/sse}",
            "timeout": 15000,
          },
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
        },
        "google-postgres": {
          "connection": {
            "endpoint": "${GOOGLE_POSTGRES_URL:-https://mcp.googleapis.com/postgres/sse}",
            "timeout": 15000,
          },
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
        },
        "google-maps": {
          "connection": {
            "endpoint": "${GOOGLE_MAPS_URL:-https://mcp.googleapis.com/maps/sse}",
            "timeout": 15000,
          },
          "auth": {
            "type": "env",
            "envName": "GOOGLE_MAPS_API_KEY",
          },
        },
        "google-search": {
          "connection": {
            "endpoint": "${GOOGLE_SEARCH_URL:-https://mcp.googleapis.com/search/sse}",
            "timeout": 15000,
          },
        },
        "qdrant": {
          "connection": {
            "endpoint": "${QDRANT_URL:-https://mcp.qdrant.tech/sse}",
            "timeout": 15000,
          },
          "auth": {
            "type": "env",
            "envName": "QDRANT_API_KEY",
          },
        },
      },
    },
    "staging": {
      "global": {
        "version": "2.0.0",
        "defaultTimeout": "${MCP_DEFAULT_TIMEOUT:-30000}",
        "defaultRetry": {
          "attempts": 3,
          "delay": 1000,
          "backoff": "exponential",
        },
        "healthMonitoring": {
          "enabled": true,
          "interval": 60000,
          "alertThreshold": 3,
        },
        "logging": {
          "level": "info",
          "destination": "gcp-logging",
          "includeRequestResponse": false,
          "structured": true,
        },
        "security": {
          "enableTLS": true,
          "validateCertificates": true,
          "maxRequestSize": 20971520,
          "rateLimiting": {
            "enabled": true,
            "requestsPerMinute": 1000,
            "burstLimit": 100,
          },
        },
        "errorHandling": {
          "gracefulDegradation": true,
          "fallbackServers": ["memory", "time", "filesystem"],
          "maxConcurrentFailures": 3,
        },
      },
      "enabledServers": [
        "semgrep",
        "nx-mcp",
        "google-filesystem",
        "google-postgres",
        "google-maps",
        "google-search",
        "qdrant",
        "filesystem",
        "git",
        "github",
        "memory",
        "sequentialthinking",
        "time",
        "fetch",
        "databases",
        "nx",
        "node",
        "google",
        "exa",
        "apimatic",
        "builtwith",
        "toolbox",
        "cloudrun",
      ],
      "serverOverrides": {
        "github": {
          "connection": {
            "timeout": 20000,
            "retry": {
              "attempts": 3,
              "delay": 1000,
              "backoff": "exponential",
            },
          },
          "healthCheck": {
            "interval": 120000,
            "failureThreshold": 3,
          },
          "auth": {
            "type": "secret-manager",
            "secretName": "github-token-staging",
          },
        },
        "databases": {
          "options": {
            "bigqueryProjectId": "${GCP_PROJECT_ID}",
            "datasets": ["analytics_staging", "signals_staging", "events_staging"],
          },
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
        },
        "google": {
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "options": {
            "services": ["bigquery", "storage", "secretmanager", "aiplatform"],
          },
        },
        "exa": {
          "auth": {
            "type": "secret-manager",
            "secretName": "exa-api-key-staging",
          },
        },
        "toolbox": {
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "connection": {
            "timeout": 45000,
          },
          "options": {
            "toolsConfigSecret": "toolbox-tools-config-staging",
          },
        },
        "cloudrun": {
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "options": {
            "invokerServiceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "connection": {
            "timeout": 45000,
          },
        },
        "semgrep": {
          "connection": {
            "endpoint": "${SEMGREP_URL:-https://mcp.semgrep.ai/sse}",
            "timeout": 15000,
          },
        },
        "nx-mcp": {
          "connection": {
            "endpoint": "${NX_MCP_URL:-https://mcp.nx.app/sse}",
            "timeout": 15000,
          },
        },
        "google-filesystem": {
          "connection": {
            "endpoint": "${GOOGLE_FILESYSTEM_URL:-https://mcp.googleapis.com/filesystem/sse}",
            "timeout": 15000,
          },
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
        },
        "google-postgres": {
          "connection": {
            "endpoint": "${GOOGLE_POSTGRES_URL:-https://mcp.googleapis.com/postgres/sse}",
            "timeout": 15000,
          },
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
        },
        "google-maps": {
          "connection": {
            "endpoint": "${GOOGLE_MAPS_URL:-https://mcp.googleapis.com/maps/sse}",
            "timeout": 15000,
          },
          "auth": {
            "type": "env",
            "envName": "GOOGLE_MAPS_API_KEY",
          },
        },
        "google-search": {
          "connection": {
            "endpoint": "${GOOGLE_SEARCH_URL:-https://mcp.googleapis.com/search/sse}",
            "timeout": 15000,
          },
        },
        "qdrant": {
          "connection": {
            "endpoint": "${QDRANT_URL:-https://mcp.qdrant.tech/sse}",
            "timeout": 15000,
          },
          "auth": {
            "type": "env",
            "envName": "QDRANT_API_KEY",
          },
        },
      },
    },
    "production": {
      "global": {
        "version": "2.0.0",
        "defaultTimeout": "${MCP_DEFAULT_TIMEOUT:-45000}",
        "defaultRetry": {
          "attempts": 5,
          "delay": 2000,
          "backoff": "exponential",
        },
        "healthMonitoring": {
          "enabled": true,
          "interval": 300000,
          "alertThreshold": 5,
        },
        "logging": {
          "level": "warn",
          "destination": "gcp-logging",
          "includeRequestResponse": false,
          "structured": true,
          "sampling": {
            "enabled": true,
            "rate": 0.1,
          },
        },
        "security": {
          "enableTLS": true,
          "validateCertificates": true,
          "maxRequestSize": 10485760,
          "rateLimiting": {
            "enabled": true,
            "requestsPerMinute": 500,
            "burstLimit": 50,
          },
          "ipWhitelist": {
            "enabled": true,
            "allowedRanges": ["10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16"],
          },
        },
        "errorHandling": {
          "gracefulDegradation": true,
          "fallbackServers": ["memory", "time"],
          "maxConcurrentFailures": 5,
          "circuitBreaker": {
            "enabled": true,
            "failureThreshold": 10,
            "recoveryTimeout": 60000,
          },
        },
      },
      "enabledServers": [
        "semgrep",
        "nx-mcp",
        "google-filesystem",
        "google-postgres",
        "google-maps",
        "google-search",
        "qdrant",
        "filesystem",
        "git",
        "github",
        "memory",
        "sequentialthinking",
        "time",
        "fetch",
        "databases",
        "nx",
        "node",
        "google-cloud",
        "exa",
        "builtwith",
        "toolbox",
        "cloudrun",
      ],
      "serverOverrides": {
        "github": {
          "connection": {
            "timeout": 45000,
            "retry": {
              "attempts": 5,
              "delay": 2000,
              "backoff": "exponential",
            },
          },
          "healthCheck": {
            "interval": 300000,
            "failureThreshold": 5,
          },
          "auth": {
            "secretName": "github-token-prod",
          },
        },
        "databases": {
          "options": {
            "bigqueryProjectId": "${GCP_PROJECT_ID}",
            "datasets": ["analytics", "signals", "events"],
          },
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "connection": {
            "timeout": 60000,
            "poolSize": 10,
          },
        },
        "google": {
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "options": {
            "services": ["bigquery", "storage", "secretmanager", "aiplatform", "cloudrun"],
          },
          "connection": {
            "timeout": 60000,
          },
        },
        "exa": {
          "auth": {
            "secretName": "exa-api-key-prod",
          },
          "connection": {
            "timeout": 30000,
          },
        },
        "toolbox": {
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "connection": {
            "timeout": 60000,
          },
          "options": {
            "toolsConfigSecret": "toolbox-tools-config-prod",
          },
        },
        "cloudrun": {
          "auth": {
            "type": "wif",
            "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "options": {
            "invokerServiceAccount": "${WIF_SERVICE_ACCOUNT}",
          },
          "connection": {
            "timeout": 60000,
          },
        },
      },
    },
    "test": {
      "global": {
        "version": "2.0.0",
        "defaultTimeout": 5000,
        "defaultRetry": {
          "attempts": 1,
          "delay": 100,
          "backoff": "linear",
        },
        "healthMonitoring": {
          "enabled": false,
          "interval": 0,
          "alertThreshold": 0,
        },
        "logging": {
          "level": "error",
          "destination": "console",
          "includeRequestResponse": false,
          "structured": false,
        },
        "security": {
          "enableTLS": false,
          "validateCertificates": false,
          "maxRequestSize": 1048576,
          "rateLimiting": {
            "enabled": false,
          },
        },
        "errorHandling": {
          "gracefulDegradation": false,
          "fallbackServers": [],
          "maxConcurrentFailures": 1,
        },
      },
      "enabledServers": ["memory", "time", "filesystem"],
      "serverOverrides": {
        "memory": {
          "connection": {
            "timeout": 1000,
          },
        },
        "time": {
          "connection": {
            "timeout": 1000,
          },
        },
        "filesystem": {
          "connection": {
            "timeout": 2000,
          },
          "options": {
            "allowedDirectories": ["/tmp", "./test-fixtures"],
          },
        },
      },
    },
  },
  "integrations": {
    "gcp": {
      "enabled": true,
      "projectId": "${GCP_PROJECT_ID}",
      "region": "${GCP_REGION:-us-central1}",
      "services": ["bigquery", "storage", "secretmanager", "aiplatform", "cloudrun"],
      "auth": {
        "type": "wif",
        "provider": "${WIF_PROVIDER}",
        "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
      },
      "secretManager": {
        "enabled": true,
        "prefix": "mcp-${NODE_ENV:-development}",
        "caching": {
          "enabled": true,
          "ttl": 3600,
        },
      },
    },
    "nx": {
      "enabled": true,
      "workspaceRoot": ".",
      "configPath": "nx.json",
      "cacheDirectory": ".nx/cache",
    },
    "github": {
      "enabled": true,
      "owner": "${GITHUB_OWNER}",
      "repo": "${GITHUB_REPO}",
      "defaultBranch": "main",
      "auth": {
        "type": "secret-manager",
        "secretName": "github-token-${NODE_ENV:-development}",
      },
    },
  },
  "monitoring": {
    "healthChecks": {
      "enabled": true,
      "interval": "${HEALTH_CHECK_INTERVAL:-60000}",
      "timeout": "${HEALTH_CHECK_TIMEOUT:-10000}",
      "retries": 3,
      "endpoints": ["/health", "/ready", "/metrics"],
    },
    "metrics": {
      "enabled": true,
      "collectInterval": "${METRICS_INTERVAL:-30000}",
      "retentionDays": "${METRICS_RETENTION:-7}",
      "exporters": ["prometheus", "gcp-monitoring"],
    },
    "alerts": {
      "enabled": true,
      "channels": ["gcp-logging"],
      "thresholds": {
        "errorRate": "${ALERT_ERROR_RATE:-5}",
        "responseTime": "${ALERT_RESPONSE_TIME:-30000}",
        "consecutiveFailures": "${ALERT_CONSECUTIVE_FAILURES:-3}",
        "memoryUsage": 85,
        "cpuUsage": 80,
      },
      "escalation": {
        "enabled": true,
        "levels": [
          {
            "threshold": 3,
            "channels": ["gcp-logging"],
          },
          {
            "threshold": 5,
            "channels": ["gcp-logging"],
          },
          {
            "threshold": 10,
            "channels": ["gcp-logging", "pagerduty"],
          },
        ],
      },
    },
    "tracing": {
      "enabled": true,
      "provider": "gcp-trace",
      "samplingRate": "${TRACE_SAMPLING_RATE:-0.1}",
    },
  },
  "security": {
    "authentication": {
      "required": true,
      "methods": ["wif", "gcp-service-account"],
      "wif": {
        "enabled": true,
        "provider": "${WIF_PROVIDER}",
        "serviceAccount": "${WIF_SERVICE_ACCOUNT}",
        "tokenLifetime": 3600,
      },
      "secretManager": {
        "enabled": true,
        "prefix": "mcp-${NODE_ENV:-development}",
        "projectId": "${GCP_PROJECT_ID}",
        "caching": {
          "enabled": true,
          "ttl": 1800,
        },
      },
    },
    "authorization": {
      "enabled": true,
      "provider": "gcp-iam",
      "roles": {
        "mcp-user": {
          "permissions": ["filesystem.read", "git.status", "memory.get", "time.get", "fetch.get"],
        },
        "mcp-developer": {
          "permissions": [
            "filesystem.read",
            "filesystem.write",
            "git.status",
            "git.commit",
            "memory.get",
            "memory.set",
            "time.get",
            "fetch.get",
            "fetch.post",
            "nx.run",
            "node.execute",
            "search.files",
            "filesystem.list",
          ],
        },
        "mcp-admin": {
          "permissions": ["*"],
        },
      },
      "defaultRole": "mcp-user",
    },
    "encryption": {
      "enabled": true,
      "provider": "gcp-kms",
      "keyRing": "mcp-encryption",
      "location": "${GCP_REGION:-us-central1}",
    },
    "audit": {
      "enabled": true,
      "destination": "gcp-audit-logs",
      "events": ["auth", "config-change", "server-start", "server-stop", "error"],
    },
  },

  "serverCatalog": [
    {
      "id": "everything",
      "name": "Everything (reference)",
      "repo": "https://github.com/modelcontextprotocol/servers/tree/main/src/everything",
      "category": "reference",
      "description": "Reference/test server with prompts, resources, and tools. Good for functional testing and examples.",
      "recommendedAuth": "none",
      "notes": "Useful for local development and example tooling.",
    },
    {
      "id": "fetch",
      "name": "Fetch",
      "repo": "https://github.com/modelcontextprotocol/servers/tree/main/src/fetch",
      "category": "reference",
      "description": "Web content fetching and conversion for LLM use.",
      "recommendedAuth": "none",
      "notes": "Consider URL allowlists and response size limits for security.",
    },
    {
      "id": "filesystem",
      "name": "Filesystem",
      "repo": "https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem",
      "category": "reference",
      "description": "Secure file operations with configurable access controls.",
      "recommendedAuth": "service-account or secret-manager",
      "notes": "Use strict allowedDirectories and careful ACLs when enabling.",
    },
    {
      "id": "git",
      "name": "Git",
      "repo": "https://github.com/modelcontextprotocol/servers/tree/main/src/git",
      "category": "reference",
      "description": "Tools to read, search, and manipulate Git repositories.",
      "recommendedAuth": "ssh-key or token",
      "notes": "Limit write operations in production.",
    },
    {
      "id": "memory",
      "name": "Memory",
      "repo": "https://github.com/modelcontextprotocol/servers/tree/main/src/memory",
      "category": "reference",
      "description": "Knowledge graph-based persistent memory system.",
      "recommendedAuth": "api-key or secret-manager",
      "notes": "Ensure encryption at rest for persisted stores.",
    },
    {
      "id": "sequentialthinking",
      "name": "Sequential Thinking",
      "repo": "https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking",
      "category": "reference",
      "description": "Dynamic and reflective problem-solving through thought sequences.",
      "recommendedAuth": "none",
      "notes": "Used for advanced agent reasoning workflows.",
    },
    {
      "id": "time",
      "name": "Time",
      "repo": "https://github.com/modelcontextprotocol/servers/tree/main/src/time",
      "category": "reference",
      "description": "Time and timezone conversion capabilities.",
      "recommendedAuth": "none",
      "notes": "Low-risk utility server.",
    },
    {
      "id": "apify",
      "name": "Apify",
      "repo": "https://github.com/apify/actors-mcp-server",
      "category": "third-party",
      "description": "Actors MCP Server for large-scale web scraping and extraction.",
      "recommendedAuth": "api-key",
      "notes": "Useful for crawling external menus and pricing data. Consider rate limits and robots.txt.",
    },
    {
      "id": "vectorstores",
      "name": "Vector Stores (pinecone/qdrant/chroma)",
      "repo": "https://github.com/modelcontextprotocol/servers (various community repos)",
      "category": "third-party",
      "description": "Vector DB adapters for semantic search (Pinecone, Qdrant, Chroma, etc.).",
      "recommendedAuth": "api-key",
      "notes": "Add specific server only after picking a provider; ensure index access controls.",
    },
    {
      "id": "meilisearch",
      "name": "Meilisearch",
      "repo": "https://github.com/meilisearch/meilisearch-mcp",
      "category": "third-party",
      "description": "Full-text & semantic search integration.",
      "recommendedAuth": "api-key",
      "notes": "Good for fast site search and product/menu search.",
    },
    {
      "id": "redis",
      "name": "Redis",
      "repo": "https://github.com/redis/mcp-redis/",
      "category": "third-party",
      "description": "Redis key-value store integration and Redis Cloud API.",
      "recommendedAuth": "api-key or managed-provider-credentials",
      "notes": "Useful for caching and ephemeral memory layers. Ensure network ACLs.",
    },
    {
      "id": "toolbox",
      "name": "MCP Toolbox (googleapis/genai-toolbox)",
      "repo": "https://github.com/googleapis/genai-toolbox",
      "category": "google",
      "description": "Open-source MCP server to safely expose database tools on GCP (BigQuery, AlloyDB, etc.).",
      "recommendedAuth": "wif / service-account",
      "notes": "Configure tools.yaml and prefer IAM/WIF for DB access.",
    },
    {
      "id": "cloudrun",
      "name": "Cloud Run MCP (GoogleCloudPlatform/cloud-run-mcp)",
      "repo": "https://github.com/GoogleCloudPlatform/cloud-run-mcp",
      "category": "google",
      "description": "Reference MCP server and deployment pattern for hosting MCP servers on Cloud Run.",
      "recommendedAuth": "gcp-iap or wif",
      "notes": "Use Cloud Run and IAM to host MCP implementations.",
    },
    {
      "id": "google-cloud",
      "name": "Google Cloud",
      "repo": "https://github.com/googleapis/google-cloud-node",
      "category": "google",
      "description": "General Google Cloud services.",
      "recommendedAuth": "wif / service-account",
    },
    {
      "id": "bq",
      "name": "BigQuery",
      "repo": "https://github.com/googleapis/google-cloud-node/tree/main/packages/google-cloud-bigquery",
      "category": "google",
      "description": "BigQuery for data warehousing and analytics.",
      "recommendedAuth": "wif / service-account",
    },
    {
      "id": "nx-mcp",
      "name": "Nx MCP",
      "repo": "npm:nx-mcp",
      "category": "third-party",
      "description": "Nx monorepo context (project graph, generators, docs)",
      "recommendedAuth": "none",
    },
    {
      "id": "google-filesystem",
      "name": "Google Filesystem",
      "repo": "https://github.com/modelcontextprotocol/server-filesystem",
      "category": "google",
      "description": "Gemini CLI MCP for filesystem access",
      "recommendedAuth": "service-account or secret-manager",
    },
    {
      "id": "google-postgres",
      "name": "Google PostgreSQL",
      "repo": "https://github.com/modelcontextprotocol/server-postgres",
      "category": "google",
      "description": "Gemini CLI MCP for PostgreSQL access",
      "recommendedAuth": "service-account or wif",
    },
    {
      "id": "google-maps",
      "name": "Google Maps",
      "repo": "docker:mcp/google-maps",
      "category": "google",
      "description": "Google Maps services (geocode, directions, places)",
      "recommendedAuth": "api-key",
      "notes": "Requires GOOGLE_MAPS_API_KEY environment variable.",
    },
    {
      "id": "google-search",
      "name": "Google Search",
      "repo": "https://github.com/mcp-server/google-search-mcp",
      "category": "google",
      "description": "Structured Google Search via browser automation",
      "recommendedAuth": "none",
    },
    {
      "id": "qdrant",
      "name": "Qdrant",
      "repo": "https://github.com/modelcontextprotocol/server-qdrant",
      "category": "third-party",
      "description": "Semantic memory store (Qdrant vector DB)",
      "recommendedAuth": "api-key",
    },
    {
      "id": "csharp-sdk",
      "name": "C# SDK",
      "repo": "nuget:MCP.Sdk",
      "category": "sdk",
      "description": "C# SDK for MCP. Stable and well-supported.",
      "recommendedAuth": "api-key",
    },
    {
      "id": "java-sdk",
      "name": "Java SDK",
      "repo": "https://github.com/spring-projects/spring-ai",
      "category": "sdk",
      "description": "Java SDK for MCP, maintained by Spring AI. Supports sync & async, multiple transports.",
      "recommendedAuth": "api-key",
    },
    {
      "id": "python-sdk",
      "name": "Python SDK",
      "repo": "pypi:mcp-sdk",
      "category": "sdk",
      "description": "Python SDK for MCP. Async-first, asyncio-based.",
      "recommendedAuth": "api-key",
    },
    {
      "id": "typescript-sdk",
      "name": "TypeScript SDK",
      "repo": "npm:@mcp/sdk",
      "category": "sdk",
      "description": "TypeScript SDK for MCP. Promise-based, Express-like API for browser & Node.js.",
      "recommendedAuth": "api-key",
    },
    {
      "id": "kotlin-sdk",
      "name": "Kotlin SDK",
      "repo": "maven:com.mcp.sdk:kotlin-sdk",
      "category": "sdk",
      "description": "Kotlin SDK for MCP. Coroutine-based, JDK 8 compatible.",
      "recommendedAuth": "api-key",
    },
    {
      "id": "go-sdk",
      "name": "Go SDK",
      "repo": "go-mcp",
      "category": "sdk",
      "description": "Go Community SDK for MCP",
      "recommendedAuth": "api-key",
    },
    {
      "id": "rust-sdk",
      "name": "Rust SDK",
      "repo": "crate:MCPR",
      "category": "sdk",
      "description": "Rust Community SDK for MCP",
      "recommendedAuth": "api-key",
    },
    {
      "id": "semgrep",
      "name": "Semgrep",
      "repo": "https://github.com/returntocorp/semgrep",
      "category": "third-party",
      "description": "Semgrep code analysis and security scanning",
      "recommendedAuth": "api-key",
    },
  ],
  "validation": {
    "schema": {
      "enabled": true,
      "strict": true,
    },
    "environmentVariables": {
      "required": ["GCP_PROJECT_ID", "WIF_PROVIDER", "WIF_SERVICE_ACCOUNT"],
      "optional": [
        "NODE_ENV",
        "GCP_REGION",
        "GITHUB_OWNER",
        "GITHUB_REPO",
        "MCP_DEFAULT_TIMEOUT",
        "HEALTH_CHECK_INTERVAL",
        "METRICS_INTERVAL",
        "SEMGREP_URL",
        "NX_MCP_URL",
        "GOOGLE_FILESYSTEM_URL",
        "GOOGLE_POSTGRES_URL",
        "GOOGLE_MAPS_URL",
        "GOOGLE_SEARCH_URL",
        "QDRANT_URL",
        "GOOGLE_MAPS_API_KEY",
        "QDRANT_API_KEY",
        "TRACE_SAMPLING_RATE",
      ],
    },
  },
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/redrover13)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/redrover13)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
