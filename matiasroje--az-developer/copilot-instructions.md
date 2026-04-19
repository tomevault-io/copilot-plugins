## az-developer

> Build simple microservices demonstrating ALL AZ-204 exam objectives.

# Azure Developer Certification Suite - AZ-204 FOCUSED

## Project Mission - AZ-204 EXAM PREPARATION

Build simple microservices demonstrating ALL AZ-204 exam objectives.
**Priority: Pass AZ-204 first, then enhance for AZ-400**

## AZ-204 Learning Strategy

**Local Development → Azure Adaptation → Azure Native**

1. **Phase 1**: Microservices with minikube (PostgreSQL + RabbitMQ + custom gateway)
2. **Phase 2**: Azure Functions adaptation (serverless patterns)
3. **Phase 3**: Azure container services (AKS/Container Apps)

## Code Philosophy - EXAM FOCUSED

- **Minimal viable code** that demonstrates exam concepts
- **Working examples** over production complexity
- **Free tier only** - immediate deletion after testing
- **Minikube first** - then Azure adaptation

## AZ-204 Exam Service Coverage

**MUST IMPLEMENT for exam:**

- ✅ **Azure Functions**: HTTP triggers, timer triggers, blob triggers
- ✅ **Container Apps**: Microservices deployment patterns
- ✅ **Blob Storage**: Image upload/processing, access tiers, SAS tokens
- ✅ **PostgreSQL**: Database operations, connection strings, firewall rules
- ✅ **Service Bus**: Queues, topics, message patterns (RabbitMQ locally)
- ✅ **Key Vault**: Secret management, Managed Identity patterns
- ✅ **API Management**: Gateway policies, rate limiting, versioning
- ✅ **Static Web Apps**: React deployment, CI/CD integration
- ✅ **Application Insights**: Logging, monitoring, custom telemetry

## Development Approach

**Local First (minikube):**

- PostgreSQL container for auth-service
- RabbitMQ container for messaging
- Custom API Gateway (FastAPI)
- React frontend
- Python microservices (FastAPI)

**Azure Adaptation:**

- PostgreSQL Flexible Server (free tier)
- Azure Service Bus (free tier)
- Azure Functions (consumption plan)
- Azure Blob Storage
- Azure Container Apps

## Code Standards - SIMPLE

- **Environment variables** for all configuration
- **Direct database connections** in development
- **Basic error handling** - just enough to understand patterns
- **Minimal logging** - console.log/print statements are fine
- **No complex middleware** unless it's an exam pattern

## Exam Explanations Required

For every Azure service implementation:

1. **Why this service?** (exam scenario context)
2. **Key exam points** (pricing, scaling, limitations)
3. **Common gotchas** (firewall rules, connection strings, etc.)
4. **Alternative services** (when to use what)

**AZ-400 comes AFTER passing AZ-204** - focus only on AZ-204 concepts until exam completion.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MatiasRoje) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
