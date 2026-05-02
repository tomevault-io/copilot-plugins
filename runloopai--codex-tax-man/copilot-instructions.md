## codex-tax-man

> This document describes the specialized tax preparation agent system built on the OpenAI Codex SDK platform.

# Tax Agent Documentation

This document describes the specialized tax preparation agent system built on the OpenAI Codex SDK platform.

## Overview

The Tax Preparation Agent provides an intelligent, end-to-end tax filing system that automatically processes tax documents, performs calculations, and generates completed tax returns. The system uses an **agent-driven architecture** where the Codex agent orchestrates tax processing via CLI tools.

### Key Components

- **Tax Agent Server**: Express-based server with agent-driven tax processing
- **taxctl CLI**: Command-line tools the agent uses to parse, calculate, and generate forms
- **AI Tax Assistant**: Conversational interface for tax guidance
- **Tax Engine**: 2024 tax calculation and Form 1040 generation

## Agent-Driven Architecture

The system uses an "agent as orchestrator" pattern where the Codex agent executes shell commands to process taxes:

```
Frontend Upload → POST /api/tax/process-documents
                      ↓
              CodexService.runAgentTurn(prompt)
                      ↓
              Agent executes CLI commands:
                - taxctl list-documents
                - taxctl parse <file>
                - taxctl calculate (JSON stdin)
                - taxctl generate-pdf (JSON stdin or file)
                      ↓
              Agent emits structured JSON result
                      ↓
              Return to frontend
```

### Why Agent-Driven?

- **Flexible**: Agent can reason about complex tax scenarios
- **Extensible**: Adding new document types (1099s, 1098s) just requires updating the prompt
- **Intelligent**: Agent can handle edge cases and ambiguous data
- **Future-proof**: Can incorporate conversational data from chat into tax processing

## taxctl CLI Tool

The `taxctl` CLI provides 4 commands the agent uses for tax processing:

```bash
# List documents in input folder
taxctl list-documents
# Output: { "success": true, "documents": [...], "count": 1 }

# Parse a tax document
taxctl parse w2_2024.pdf
# Output: { "success": true, "documentType": "w2", "data": {...} }

# Calculate taxes (reads JSON from stdin)
echo '{"w2Forms": [...], "taxpayerInfo": {...}, "filingStatus": "single"}' | taxctl calculate
# Output: { "success": true, "form1040": {...}, "summary": {...} }

# Generate PDF (from file or stdin)
taxctl generate-pdf /tmp/content/output/form1040.json
# OR pipe from calculate:
echo '{"form1040": {...}}' | taxctl generate-pdf
# Output: { "success": true, "pdfPath": "/tmp/content/output/....pdf" }
```

### Agent Package File Structure

```
packages/tax-processing/src/
  cli/
    index.ts                    # Commander CLI entry point
    commands/
      list-documents.ts         # List /tmp/content/input/ files
      parse.ts                  # Parse W-2/1099 documents
      calculate.ts              # Calculate taxes (stdin JSON)
      generate-pdf.ts           # Generate PDF from JSON (file or stdin)
  bin/
    taxctl.ts                   # Executable entry point
    run-agent-turn.ts           # Single-turn agent runner
  lib/
    agent-loader.ts             # Prompt loading infrastructure
    tax-agent-library.ts        # Tax-specific agent API
  prompts/
    agent.txt                   # Main agent behavior
    shared/
      w2-parsing.txt            # W-2 parsing instructions
      tax-brackets-2024.txt     # Tax calculation rules (2024)
      form1040-schema.txt       # Form 1040 JSON schema
  services/
    codex.service.ts            # Codex SDK wrapper
    document-parser.ts          # W-2 parsing logic
    tax-engine.ts               # Tax calculations
    pdf-generator.ts            # PDF generation
```

## Core Services

### CodexService (`packages/tax-processing/src/services/codex.service.ts`)

The enhanced `CodexService` provides two modes:

1. **Streaming Chat** (`streamResponse`): For conversational tax guidance via SSE
2. **Agent Turn** (`runAgentTurn`): For single-turn agent processing with CLI tools

```typescript
// Agent-driven processing
const result = await codexService.runAgentTurn(prompt);
// Returns: { success, response, rawResponse, error? }
```

### Tax Agent Prompt Library (`packages/tax-processing/src/lib/tax-agent-library.ts`)

The tax agent prompt system is now modularized using text files for easy customization:

**Main Agent File** (`src/prompts/agent.txt`):
- High-level agent behavior and workflow
- References to shared instruction files
- Critical rules and error handling

**Shared Instruction Files** (`src/prompts/shared/`):
- `w2-parsing.txt`: W-2 parsing instructions and CLI tool usage
- `tax-brackets-2024.txt`: 2024 tax calculation workflow and brackets
- `form1040-schema.txt`: Complete Form 1040 JSON schema

**API**:
```typescript
// Load complete tax agent system prompt
const systemPrompt = getTaxAgentSystemPrompt();

// Build task-specific prompt with context
const taskPrompt = buildTaxAgentPrompt({
  filename: 'w2_2024.pdf',
  filingStatus: 'single',
  taxpayerInfo: { ... }
});

// Use custom agent file for benchmarking
const customPrompt = buildCustomTaxAgentPrompt(
  'examples/agent-conservative.txt',
  { filename: 'w2_2024.pdf', filingStatus: 'single' }
);
```

### Supporting Services

- **DocumentParserService** (`services/document-parser.ts`): W-2 parsing with pattern recognition
- **TaxCalculationEngine** (`services/tax-engine.ts`): 2024 tax calculations
- **PDFGeneratorService** (`services/pdf-generator.ts`): Form 1040 PDF generation

## API Endpoints

### Tax Processing

**POST `/api/tax/process-documents`** - Agent-driven document processing

```json
{
  "filename": "w2_2024.pdf",
  "taxpayerInfo": {
    "firstName": "John",
    "lastName": "Doe",
    "ssn": "XXX-XX-1234",
    "address": "123 Main St",
    "city": "San Francisco",
    "state": "CA",
    "zipCode": "94102"
  },
  "filingStatus": "single"
}
```

**POST `/api/tax/upload`** - Upload tax documents

**GET `/api/tax/download/:filename`** - Download generated forms

### Chat

**POST `/api/codex/chat`** - Conversational tax assistance (SSE stream)

**DELETE `/api/codex/thread/:threadId`** - Clean up conversation


## Tax Agent Capabilities

### Document Processing
- **W-2 Parsing**: Extracts wages, withholdings, employer info via pattern recognition
- **Multi-format Support**: PDF (via pdf-parse) and text files
- **Extensible**: Architecture supports adding 1099-INT, 1099-DIV, 1099-MISC, etc.

### Tax Calculations (2024)
- **AGI Calculation**: Adjusted Gross Income from all income sources
- **Standard Deduction**: Single $14,600, MFJ $29,200
- **Tax Brackets**: 10%, 12%, 22%, 24%, 32%, 35%, 37%
- **Credits**: Child tax credit ($2,000/child), EITC
- **Refund/Owed**: Calculates final refund or amount owed

### Form Generation
- **Form 1040 PDF**: Fills official IRS Form 1040 using pdf-lib
- **JSON Output**: Structured Form1040Data for programmatic access

## Configuration

### Sandbox Configuration

Sandbox permissions are configured in code via `CodexService.runAgentTurn()`:

```typescript
const thread = this.codex.startThread({
  workingDirectory: process.cwd(),
  networkAccessEnabled: true,
  sandboxMode: 'workspace-write',
  approvalPolicy: 'never',
  additionalDirectories: ['/tmp', '/tmp/content', '/tmp/content/output', '/tmp/content/input'],
});
```

This ensures the agent can write to `/tmp` for tax document processing without requiring user configuration.

### Environment Variables

**Agent Server (`packages/tax-processing/.env`) - Local Development Only:**
```env
PORT=3001
FRONTEND_URL=http://localhost:3000
OPENAI_API_KEY=your-api-key-here

# Optional: Weights & Biases Weave for LLM tracing (local dev only)
WANDB_API_KEY=your-wandb-api-key-here
```

**Note**: When running on Runloop devboxes, API keys must be configured as Runloop secrets, not local environment variables. The `OPENAI_API_KEY` and `WANDB_API_KEY` secrets are automatically injected into devboxes from the Runloop secret store.

**Frontend (`packages/frontend/.env.local`):**
```env
NEXT_PUBLIC_AGENT_URL=http://localhost:3001

# Cloud mode configuration
RUN_MODE=local                    # or "cloud" for Runloop
RUNLOOP_API_KEY=your-key-here     # Required for cloud mode
RUNLOOP_BASE_URL=https://api.runloop.ai
GITHUB_TOKEN=your-token-here      # Required for private repos
```

### LLM Observability with Weave

The tax agent integrates with [Weights & Biases Weave](https://docs.wandb.ai/weave/) for comprehensive LLM call tracing and monitoring.

**Setup for Runloop Devboxes:**
1. Get your W&B API key from https://wandb.ai/authorize
2. Add `WANDB_API_KEY` to Runloop secrets:
   - Option A: Use the Runloop Settings page at https://platform.runloop.ai/settings
   - Option B: Run `pnpm step1_runloop_setup` - it will prompt you to add the secret
3. When the agent runs on a devbox, Weave initializes automatically if the secret is present

**Setup for Local Development:**
1. Get your W&B API key from https://wandb.ai/authorize
2. Add `WANDB_API_KEY=your-key` to `packages/tax-processing/.env`
3. Start the agent server - Weave initializes automatically

**What Weave Tracks:**
- All OpenAI API calls made through the Codex SDK
- Complete input prompts and output responses
- Token usage, latency, and cost metrics
- Error traces and debugging information
- Agent tool usage (taxctl commands)

**Viewing Traces:**
- Navigate to https://wandb.ai/
- Select project: `tax-preparation-agent`
- Explore traces, latency distributions, and token usage

**Configuration:**
The CodexService automatically initializes Weave if `WANDB_API_KEY` is present in the environment (from Runloop secrets on devboxes, or from local `.env` file). Look for:
- `[CodexService] Weave tracing initialized successfully` - Weave is active
- `[CodexService] Weave tracing disabled: WANDB_API_KEY not set` - Running without Weave

Weave integration is completely optional - the system functions normally without it.

## Tax Agent Prompt Architecture

The tax agent uses a modular prompt system with text files for easy customization and benchmarking.

### Prompt File Structure

```
packages/tax-processing/src/prompts/
  agent.txt                      # Main agent behavior and workflow
  shared/
    w2-parsing.txt              # W-2 parsing and CLI tool usage
    tax-brackets-2024.txt       # Tax calculation workflow (2024)
    form1040-schema.txt         # Form 1040 JSON schema
```

### Prompt Composition

The agent prompt is composed in layers:

1. **Main Agent File** (`agent.txt`): High-level behavior, workflow, and rules
2. **Shared Instructions**: Domain-specific knowledge appended with clear markers
3. **Task Context**: Dynamic injection of filename, filing status, taxpayer info

### Customizing Agent Behavior

To create a custom agent for benchmarking:

1. Copy `agent.txt` to `examples/agent-custom.txt`
2. Modify the agent behavior (e.g., more conservative tax strategies)
3. Use `buildCustomTaxAgentPrompt()` to load your custom agent:

```typescript
import { buildCustomTaxAgentPrompt } from './lib/tax-agent-library.js';

const prompt = buildCustomTaxAgentPrompt(
  'examples/agent-conservative.txt',
  { filename: 'w2_2024.pdf', filingStatus: 'single' }
);
```

The shared instruction files remain the same, ensuring consistent tax knowledge across all agent variants.

## Deployment Modes

### Local Development (Recommended for Tax Preparation)
```bash
pnpm dev
```
- Tax processing services run locally for data privacy
- Document parsing and calculations happen on your machine
- Tax documents remain in local `content/` directories
- Suitable for personal tax preparation

### Cloud Mode (Runloop) - Enterprise Use
```bash
export RUN_MODE=cloud
export RUNLOOP_API_KEY=your-api-key
pnpm dev:frontend
```
- **Note**: Cloud deployment moves tax documents to remote servers
- Consider data privacy and compliance requirements
- Suitable for tax professional services with proper security
- Each client gets isolated processing environment

### Production Deployment

Use the GitHub Action workflow to deploy persistent agents:
1. Configure `RUNLOOP_API_KEY` secret in repository settings
2. Trigger "Deploy to Runloop" workflow
3. Agent becomes available as named service on Runloop platform

## API Reference

### Agent Creation (Cloud Mode)
```typescript
interface CreateAgentOptions {
  workingDir?: string;  // GitHub repository URL
}

const agent = await agentManager.createAgent({
  workingDir: 'https://github.com/user/repo'
});
```

### Health Checking
```typescript
const health = await agentManager.checkHealth(agentId);
// Returns: { healthy: boolean, data?: unknown }
```

### Thread Management
```typescript
// Create new thread
POST /chat
{ "message": "Hello" }

// Continue thread  
POST /chat
{ "message": "Follow up", "threadId": "existing-id" }

// Clean up thread
DELETE /thread/thread-id
```

## Event System

The agent communication uses Server-Sent Events for real-time streaming:

### Event Types

- **`threadId`**: New conversation thread identifier
- **`delta`**: Incremental response text
- **`message`**: Complete agent message or tool output
- **`done`**: Response completion with usage statistics
- **`error`**: Error conditions and failures

### Event Processing

Frontend processes events through the `useChat` hook:
```typescript
const { messages, sendMessage, isLoading } = useChat(threadId);
```

## Security Considerations

### Sandbox Isolation
- Agents run in isolated sandbox environments
- File access limited to workspace directory
- Network access controlled via configuration

### API Key Management
- OpenAI keys stored in environment variables
- Graceful degradation when keys are missing
- No key exposure in client-side code

### Cloud Security
- HTTPS tunnels for encrypted communication
- Isolated devboxes prevent cross-contamination
- Automatic cleanup and resource limits

## Troubleshooting

### Common Issues

**CORS Errors:**
- Verify `FRONTEND_URL` matches Next.js dev server
- Check CORS middleware configuration

**Streaming Failures:**
- Ensure browser supports Server-Sent Events
- Check network proxy/firewall settings
- Verify agent server is accessible

**Cloud Deployment Issues:**
- Validate `RUNLOOP_API_KEY` and permissions
- Check GitHub repository access and `GITHUB_TOKEN`
- Monitor devbox creation and startup logs

### Debug Commands

```bash
# Check agent health
curl http://localhost:3001/health

# View agent logs
pnpm dev:agent --verbose

# Test cloud connectivity
curl -H "Authorization: Bearer $RUNLOOP_API_KEY" \
     https://api.runloop.ai/v1/devboxes
```

## Performance Optimization

### Local Mode
- Keep agent processes warm between requests
- Use thread pooling for concurrent conversations
- Monitor memory usage with long-running threads

### Cloud Mode  
- Configure appropriate keep-alive timeouts
- Use regional deployment for reduced latency
- Monitor devbox resource utilization

## Tax Workflow Example

### Agent-Driven Processing (Default)

```bash
# 1. Upload W-2 document
curl -X POST -F "document=@w2-2024.pdf" http://localhost:3001/api/tax/upload

# 2. Process with agent (all-in-one: parse → calculate → generate)
curl -X POST http://localhost:3001/api/tax/process-documents \
  -H "Content-Type: application/json" \
  -d '{
    "filename": "w2-2024_1234567890.pdf",
    "taxpayerInfo": {
      "firstName": "John",
      "lastName": "Doe",
      "ssn": "XXX-XX-1234",
      "address": "123 Main St",
      "city": "San Francisco",
      "state": "CA",
      "zipCode": "94102"
    },
    "filingStatus": "single"
  }'

# 3. Download generated PDF
curl http://localhost:3001/api/tax/download/Doe_2024_Form1040_2024-12-08.pdf -o form1040.pdf
```

### CLI Direct Usage (Testing/Development)

```bash
# List documents
node dist/bin/taxctl.js list-documents

# Parse a W-2
node dist/bin/taxctl.js parse w2_sample.txt

# Calculate taxes
echo '{"w2Forms":[...], "taxpayerInfo":{...}, "filingStatus":"single"}' | node dist/bin/taxctl.js calculate

# Generate PDF (from file or stdin - IRS template is bundled)
node dist/bin/taxctl.js generate-pdf /tmp/content/output/form1040.json
# OR pipe directly from calculate:
echo '{"form1040":{...}}' | node dist/bin/taxctl.js generate-pdf
```

## Future Tax Enhancements

### Tax Features
- [ ] State tax return generation (multiple states)
- [ ] Schedule C business income support
- [ ] Schedule D capital gains/losses
- [ ] Advanced itemized deductions
- [ ] Multi-year tax return analysis
- [ ] Tax amendment (1040X) generation
- [ ] E-filing integration with IRS systems

### Document Support
- [ ] 1099 forms (INT, DIV, MISC, NEC)
- [ ] 1098 mortgage interest statements
- [ ] Charitable contribution receipts
- [ ] Medical expense documentation
- [ ] Education expense forms (1098-T)

### Technical Improvements
- [ ] Advanced PDF parsing for complex documents
- [ ] Blockchain-based audit trails
- [ ] End-to-end encryption for cloud deployment
- [ ] Multi-tenant tax professional platform
- [ ] Integration with accounting software (QuickBooks, Xero)
- [ ] Real-time IRS API integration for current rates

---
> Source: [runloopai/codex-tax-man](https://github.com/runloopai/codex-tax-man) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
