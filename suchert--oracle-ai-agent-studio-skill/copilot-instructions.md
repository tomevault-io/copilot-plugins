## oracle-ai-agent-studio-skill

> >


# Oracle AI Agent Studio

A comprehensive skill for working with Oracle AI Agent Studio — the design-time environment
within Oracle Fusion Cloud Applications for creating, configuring, validating, and deploying
AI agents and agent teams across the enterprise.

## When to Use This Skill

- Creating new AI agents or agent teams in Oracle Fusion
- Customizing or extending pre-built Oracle agent templates
- Configuring agent tools (Document, REST, Business Object, Calculator, Email)
- Setting up multi-agent workflows with branching, chaining, and human-in-the-loop
- Connecting agents to third-party LLMs or external systems via MCP
- Testing, evaluating, and monitoring agent performance
- Deploying agents through the Oracle AI Agent Marketplace
- Integrating custom agents into Fusion Service, HCM, ERP, or SCM workflows

For Oracle Database-specific skills (SQL tuning, PL/SQL, ORDS, security audits), see
`krisrice/oracle-db-skills` instead. This skill focuses on the Fusion Applications
agent platform, not the OCI AI Database Private Agent Factory (see `references/oci-agent-factory.md`
for that).

---

## Architecture Overview

Oracle AI Agent Studio has five core components that work together hierarchically:

```
Agent Team (deployable unit)
 └── Agent(s)
      ├── Topics (expertise boundaries)
      ├── Instructions (natural language rules per topic)
      └── Tools (utilities the agent can invoke)
```

### Component Details

**Agent Team**: The deployable unit. Consists of a structured sequence of steps or actions
that one or more agents follow to accomplish a business task. Agent teams define conversation
logic, system integration, and user support flow. An agent team is what gets published and
invoked at runtime.

**Agent**: Uses an LLM to reason, plan, and interact with users. Must belong to an agent team.
Three types exist:
- *User-proxy agent*: Acts on behalf of a user, gathers input, provides conversational interface
- *Supervisor agent*: Orchestrates multiple agents within a workflow
- *Specialist/utility agent*: Focused on a specific role or tool expertise

Agents can also be persona-based (e.g., benefits administrator), tool users (calculators,
document retrieval), or task-oriented (single step in a multi-agent flow).

**Topics**: Define expertise boundaries through instructions. Example: an employee benefits
agent might have topics for HSA, retirement benefits, and stock plans. Topics set the scope
of what an agent can and cannot discuss.

**Tools**: Additional utilities an agent can invoke. Types include:
- *Document tool*: For RAG — upload documents the agent can search
- *Business Object tool*: Query/update Fusion business objects
- *REST tool*: Call external HTTPS APIs
- *Calculator tool*: Perform computations
- *Email tool*: Send notifications
- *User query tool*: Interactive data lookup
- *Agent tool*: Invoke another agent (for multi-agent collaboration)
- *MCP tool*: Connect to external systems via Model Context Protocol

**Instructions**: Natural language rules per topic. Become part of the prompt sent to the LLM.
Include guidelines, guardrails, and response parameters.

---

## Getting Started

### Prerequisites

1. Oracle Fusion Cloud Applications environment (25A or later for full Agent Studio)
2. Appropriate roles granted by an AI Studio Administrator
3. `ORA_ASE_SAS_INTEGRATION_ENABLED` profile option set to `Yes`
4. Permission groups enabled for the appropriate roles

For full access requirements, read `references/access-requirements.md`.

### Navigating to AI Agent Studio

Navigate to: **Navigator > Tools > AI Agent Studio**

The interface has four main tabs:
- **Tools** — Create and manage agent tools
- **Agents** — Create and configure individual agents
- **Agent Teams** — Assemble agents into deployable teams
- **Prompt Library** — Central store for prompts and topics

### Using the Prompt Library

Central store for managing prompts and topics across agents. Use it to save
reusable prompts shared across agents, manage topics centrally for consistent
expertise boundaries, track prompt version lifecycle (authoring → testing →
production), and discover existing topics before creating duplicates.
Access via the **Prompt Library** tab in AI Agent Studio.

---

## Creating a Custom Agent — Step by Step

Follow this sequence: Tools → Agents → Agent Teams → Test → Publish → Deploy.

### Step 1: Create Tools

1. Navigate to **Tools > AI Agent Studio**
2. Click the **Tools** tab, then **Add**
3. Select the Tool Type:
   - **Document**: Upload documents for RAG. Set status to "Ready to publish"
   - **REST**: Define an HTTPS endpoint for external API calls
   - **Business Object**: Connect to Fusion business objects
4. Configure tool-specific settings and **Save**

When creating Document tools, upload relevant knowledge base files. Supported file
types: **PDF** (tagged or scanned), **TXT**, **DOCX**, **XLSX**, **PNG**, and **JPEG**.
Users can upload up to 5 files per interaction, with a total combined size limit
of 50 MB. For OCI-based RAG data sources, additionally supported: JSON, HTML, and
Markdown (MD), with a per-file limit of 100 MB.
The agent will use these for retrieval-augmented generation.

### Step 2: Create an Agent

1. Click the **Agents** tab, then **Add**
2. Fill in agent details:
   - **Name**: Descriptive identifier (e.g., `invoice_processing_agent`)
   - **Prompt**: System prompt defining the agent's role, behavior, and task scope
   - **Topics**: Add topics to define expertise boundaries
   - **Instructions**: Write natural language rules per topic
3. Go to the **LLM** tab:
   - Select the LLM provider and model
   - For Summarization Mode: choose "Enable using custom prompt" if you need
     specific output formatting
   - Configure the Summarization Prompt for output structure
4. Attach tools: Search for tools you created, click **+**, then **Add**
5. Click **Create**

### Step 3: Assemble an Agent Team

1. Click the **Agent Teams** tab, then **Add**
2. Provide team details:
   - **Name**: Team identifier
   - **Description**: What the team accomplishes
   - **Workflow type**: Choose between conversational or workflow-based
3. Add agents: Search for agents, click **+**, then **Add**
4. For workflow-type teams, configure the execution flow in the visual designer:
   - **Add nodes**: Drag agent nodes, decision nodes, and action nodes onto the canvas
   - **Connect nodes**: Draw edges between nodes to define execution order
   - **Conditional branching**: Add decision nodes with conditions (e.g., "if refund > $500")
     that route to different agent paths based on runtime data
   - **Chaining**: Connect workflow outputs to inputs of the next step for multi-stage
     processing (e.g., Intake → Matching → Validation)
   - **Agent node**: Insert an Agent node when a step requires dynamic LLM reasoning
     rather than a deterministic action — the agent decides what to do based on context
   - **Human-in-the-loop**: Add an approval node that pauses the workflow, notifies a
     human reviewer, and resumes only after explicit approval
   - **Parallel paths**: Split execution into concurrent branches that rejoin later
5. Click **Create**

### Step 4: Test

1. Click the debug/play icon on your agent team
2. Enter test queries that represent real user scenarios
3. Verify:
   - Correct tool invocation
   - Proper topic routing
   - Accurate reasoning and sources cited
   - Appropriate guardrail enforcement

### Step 5: Publish and Deploy

1. Click **Publish** on the agent team
2. The agent team becomes available for integration
3. For Fusion Service integration, create an action type (see next section)

---

## Integrating Agents into Fusion Workflows

To make a custom agent available in Oracle Fusion Service:

### Create an Action Type

1. Navigate to **Service > Service Center Administration**
2. Click **Productivity** > **Add / Manage Workflow Action Types and Actions**
3. Click **Create Action**
4. Set **Parent Entity Name** to "Service request"
5. Click **Add Action Type** with these settings:
   - Action Type Name: e.g., `my_custom_ai_agent`
   - Application: "Oracle Fusion Applications"
   - Authentication Context: "Oracle CRM Fusion REST API"
   - ActionType: "Rest"
   - Operation: "Create"
   - Action URL: `/api/fusion-ai/orchestrator/agent/v1/<agent_team_name>/invokeAsync`
6. On the Action (Payload) page:
   - Action Name: e.g., `my_custom_agent_action`
   - Async Mode: "Async with Polling"
7. On the Polling Action page:
   - Endpoint URL: `/api/fusion-ai/orchestrator/agent/v1/<agent_team_name>/status/`
   - Request Id field: `requestId`
8. Click **Submit**

### Add to a Workflow

1. Navigate to **Service > Service Center Administration > Create/Manage Workflows**
2. Click **Create**, hover over the play icon, click **+**
3. Click **Action Types** and search for your custom action type
4. The agent is now callable like any built-in agent

---

## Workflow Architecture Patterns

Five patterns for workflow-type Agent Teams (all configurable in the visual designer):

**Deterministic Execution**: Fixed sequence with pre-defined outcomes for each input
type. Use for business-critical processes requiring consistent, repeatable results.

**Chaining**: Connect workflows end-to-end — output from one becomes input for the
next. Use for complex processes like order-to-cash or hire-to-retire.

**Agent Node**: Insert a dynamic LLM reasoning step within an otherwise deterministic
workflow. The agent decides based on context — use when a step requires interpretation.

**Human-in-the-Loop**: Approval gates that pause the workflow, notify a reviewer, and
resume after explicit approval. Essential for financial, hiring, and compliance checks.

**Supervisor + Specialist**: A supervisor agent routes queries to the appropriate
specialist. Each specialist has deep, narrow expertise. Supervisor aggregates
responses into a unified answer.

---

## LLM Configuration

Oracle AI Agent Studio supports multiple LLM providers:

| Provider | Models | Best For |
|----------|--------|----------|
| Oracle (OCI GenAI) | Cohere Command R+ | Default, no extra config |
| OpenAI | GPT-4o, GPT-4 | Complex reasoning |
| Anthropic | Claude Sonnet, Claude Opus | Nuanced analysis, long context |
| Google | Gemini | Multimodal tasks |
| Meta | Llama | Cost-efficient tasks |
| Cohere | Command R+ | RAG-optimized tasks |
| xAI | Grok | General purpose |

To configure an LLM:
1. Open your agent in the Agent Designer
2. Navigate to the **LLM** tab
3. Select the Provider from the dropdown
4. Choose the specific model
5. If the Provider list is empty, check that your environment has the required
   OCI policies and service connections configured

### Endpoint Geography and Data Residency

LLM endpoints for AI agents in Fusion Applications are hosted in specific OCI
regions. Data sent to third-party LLMs (OpenAI, Anthropic, etc.) traverses through
OCI's infrastructure to those providers' endpoints. Consider:

- **Data residency requirements**: Verify which regions host the GenAI/Agentic AI
  realm for your Fusion pod. Search Oracle Help Center for the exact title
  **"Where are the locations of the OpenAI endpoints for AI agents in Fusion
  Applications?"** (use quotes for a precise match) or see
  `references/official-docs-links.md` for the Help Center link.
- **OCI GenAI (Cohere)**: Hosted within OCI regions — data stays in Oracle's cloud
- **Third-party providers**: Data is sent to external endpoints (OpenAI US, Anthropic
  US/EU, etc.) — review your organisation's data sovereignty policies.
  **Never send PII, salary, medical, or other regulated data to a third-party LLM
  provider without a signed Data Processing Agreement (DPA) approved by your DPO.**
- **A2A and MCP calls**: External system integrations may route data outside
  your primary region — validate with your security and compliance team

For comprehensive data-classification guidance and GDPR considerations, read
`references/security-considerations.md`.

---

## Monitoring, Evaluation, and Observability

Oracle AI Agent Studio includes built-in observability:

**Agent Tracing**: Captures detailed execution data — the complete chain of reasoning,
tool calls, and responses for every interaction. Use for debugging and optimization.

**Performance Metrics**: Monitor correctness, latency, API error rates, and token
consumption. Track efficiency over time for cost management.

**Evaluation Runs**: Create test datasets with expected outcomes, run evaluation
batches, compare runs across iterations, and track improvements with versioned evals.
Supports A/B comparisons and dataset management.

**Topics Management**: Central store for all topics across agents — provides visibility
into which capabilities are assigned where, ensuring consistent prompt boundaries.

---

## Pre-Built Agent Catalog

Oracle provides 70+ pre-built agent templates across business functions.
For the full catalog organized by ERP, HCM, SCM, and CX, read
`references/prebuilt-agents-catalog.md`.

Key categories:
- **ERP/Finance**: Payables Agent, Ledger Agent, Planning Agent, Payments Agent,
  Expense Policy Agent, Profitability Agent
- **HCM**: Career Coach, Skill Recommendation Agent, Benefits Plan Advisor,
  Payslip Analyst, Leave and Absence Analyst, Onboarding Assistant,
  Employee Concierge, Manager Concierge
- **SCM/Manufacturing**: Item Shortages Advisor, Maintenance Work Order Builder,
  Operational Procedure Advisor, Purchase Requisition Status Advisor
- **CX/Service**: Knowledge Search Agent, Service Request Agent,
  Field Service Agent

To use a pre-built template:
1. Search for the template agent in AI Agent Studio
2. Click **Copy Template**
3. Enter a suffix for the agent name
4. Review and customize in the Agent Designer
5. Publish

### Using the AI Agent Marketplace

The Marketplace provides partner-built agent templates accessible directly within
AI Agent Studio. Oracle tests, approves, and supports these partner templates.

To browse and install:
1. In AI Agent Studio, go to the **Agent Teams** tab
2. Click **Marketplace** to switch from the default templates view
3. Browse or search partner templates; click the detail icon for descriptions
4. Click **Create** to install a local copy with all agents, tools, and topics
5. Customize: artifacts from templates aren't directly editable — create a copy of
   each artifact you want to modify, then swap it into the agent team
6. Test, then **Publish**

Published teams are visible to end users from the AI Agents page (add
`agent-explore` to the AI Agent Studio URL). For detailed Marketplace behaviors
and template editing rules, read `references/prebuilt-agents-catalog.md`.

---

## MCP Integration

Oracle AI Agent Studio supports Model Context Protocol (MCP) for connecting agents
to third-party data and tools. MCP enables seamless integration with external
systems without custom API development.

To add an MCP tool:
1. Create a new tool of type MCP in the Tools tab
2. Provide the MCP server URL (HTTPS only)
3. Configure authentication (do not leave unauthenticated; store credentials
   in OCI Vault or the Fusion Credentials tab — not in the tool description)
4. The agent can now discover and use tools exposed by the MCP server

This is particularly powerful for connecting Fusion agents to:
- External databases (via Oracle MCP servers at `github.com/oracle/mcp`)
- Development tools and code repositories
- Third-party SaaS applications
- Custom internal services

> **Security**: Validate that every MCP server you connect to is under your
> organisation's control or from a trusted vendor. Agents execute whatever
> tools the MCP server exposes. See `references/security-considerations.md`.

---

## A2A Integration (Agent-to-Agent Protocol)

Available from **25B (Oct 2025)**. A2A is an open protocol that allows agents
built on different platforms to collaborate across system boundaries — for example,
a Fusion agent delegating a sub-task to an agent running on a partner's platform.

### Key Concepts

- **A2A Client**: The initiating agent that delegates tasks to remote agents
- **A2A Server**: The remote agent that receives and executes delegated tasks
- **Agent Card**: A JSON descriptor (served at `/.well-known/agent.json`) that
  advertises an agent's capabilities, supported skills, and endpoint URL

### How to Use A2A in AI Agent Studio

1. **Consuming an external A2A agent** (your Fusion agent as A2A Client):
   - Create an Agent tool of type **A2A**
   - Provide the remote agent's base URL; Studio fetches the Agent Card automatically
   - Attach the tool to your agent as you would any other tool type

2. **Exposing a Fusion agent as an A2A Server**:
   - Publish the agent team; the platform automatically generates an Agent Card
     at the agent endpoint's well-known URL
   - External A2A clients can then discover and invoke your agent

### A2A vs. MCP

| Aspect | A2A | MCP |
|--------|-----|-----|
| Purpose | Agent-to-agent task delegation | Agent-to-tool/data integration |
| Counter-party | Another AI agent (LLM-based) | A server exposing tools or data |
| Discovery | Agent Card (`/.well-known/agent.json`) | MCP server manifest |
| Typical use | Cross-platform multi-agent workflows | External databases, SaaS APIs |

### Data Sovereignty Note

A2A calls may route data outside your primary OCI region or tenancy boundary.
Validate with your security and compliance team before enabling cross-tenancy
or cross-region A2A integrations that involve personal or regulated data.

---

## Profile Options Reference

Key profile options for AI Agent Studio configuration:

| Profile Option | Purpose | Values |
|----------------|---------|--------|
| `ORA_ASE_SAS_INTEGRATION_ENABLED` | Enable Security Console integration | Yes / No |
| `ORA_HRT_SKILL_SUGGESTIONS` | Control AI source for skill suggestions | Legacy / Next Generation / Next Generation with Workflow Agents |
| `ORA_WLF_AI_COMPANY_INFO` | Organization description (~1000 chars) | Free text |
| `ORA_HRT_SKILL_SUGGESTIONS_AGENT_CODE` | Link agent team to skill suggestions | Agent team code |

For HCM skill recommendation agents specifically:
1. Stop the scheduled process "Synchronize Talent Data for AI Recommendations"
2. Set `ORA_WLF_AI_COMPANY_INFO` at Site level with org description
3. Run "Create Summaries Using Generative AI" (Full first time, then Incremental)
4. Copy the Skill Recommendation Agent template
5. Publish and set the agent team code in `ORA_HRT_SKILL_SUGGESTIONS_AGENT_CODE`

---

## Security Model

AI agents in Oracle AI Agent Studio automatically inherit the security framework
of Oracle Fusion Applications:

- Agents enforce the same role-based access controls as the underlying application
- No need to reconfigure security settings or sign new agreements
- Data access is scoped to the user's permissions
- Audit trails capture all agent interactions
- Cross-tenancy agent tools require explicit IAM policies using least-privilege
  verbs (`use genai-agent-endpoints`, not `manage genai-agent-family`)

**What Fusion security does NOT cover automatically** (your responsibility):
- Prompt injection mitigations
- Credential security for external REST/MCP endpoints
- Data sovereignty when using third-party LLMs with PII or regulated data
- Audit log retention and GDPR compliance
- Input validation for agent write operations

For comprehensive security guidance including prompt injection, credential
management, data classification, and compliance considerations, read
`references/security-considerations.md`.

For cross-tenancy Agent tool configurations, read `references/cross-tenancy-policies.md`.

---

## Token Usage and Quota Management

Available from **25B**. Monitor and control LLM token consumption to manage costs
and avoid hitting service limits.

### Viewing Token Metrics
1. In AI Agent Studio, navigate to the **Performance Metrics** view
2. Filter by agent team, date range, and LLM provider
3. Review token consumption per interaction, per agent, and per provider

### Cost Control Practices
- Set a **Summarization Prompt** to constrain response length rather than
  relying on the LLM to decide verbosity
- Prefer Oracle OCI GenAI (Cohere) for high-volume, simple queries — it is
  included in the Fusion subscription without per-token cost to the tenant
- Use third-party LLMs (OpenAI, Anthropic) selectively for complex reasoning
  tasks where their quality advantage justifies the cost
- Review token metrics after the first week of production use and adjust
  the LLM provider or prompt verbosity as needed

### Rate Limits
Rate limits for OCI Generative AI Agents depend on your OCI tenancy service
limits. If agents return errors during peak load:
1. Check OCI Console > Limits, Quotas and Usage > Generative AI
2. Submit a service limit increase request if needed
3. As a temporary measure, add retry logic in the calling application

---

## Feature Availability by Release

Not all features are available in every Fusion release. Key milestones:
- **25A (Mar 2025)**: Core Agent Studio — Agent Designer, Tools, Agents, Agent Teams, testing
- **25B (Oct 2025)**: Third-party LLMs, MCP/A2A support, Marketplace, workflow agents,
  human-in-the-loop, agent tracing, performance metrics
- **25C / 26A (2026)**: Prompt Library, Topics management, runtime file upload

For the full feature-by-release matrix, read `references/release-availability.md`.

---

## Troubleshooting

### Provider list empty in LLM tab

Check that your environment has the appropriate OCI services deployed and IAM
policies configured. Verify `ORA_ASE_SAS_INTEGRATION_ENABLED` is set to `Yes`.

### Agent team not visible in Agent Teams tab

Your role must be explicitly granted access by an AI Studio Administrator.
Check permission group assignments.

### REST tool only supports HTTPS

This is by design — all external API calls must use HTTPS for security.
HTTP endpoints are not supported.

### Skills not appearing after Create Summaries job

Run the scheduled process with Run Type "Incremental" and Object Type "Skills".
New skills added to the library require this job to be processed.

### Agent returning incorrect responses

1. Check topic boundaries — ensure instructions are specific enough
2. Verify tool configurations — especially document tool content freshness
3. Review agent tracing for the reasoning chain
4. Test with different LLM providers if the issue is model-specific
5. Check that guardrails in instructions are clear and unambiguous

### Rollback After a Failed or Degraded Deployment

If a newly published agent team causes regressions or unexpected behaviour in
production:

1. **Un-publish the agent team** in AI Agent Studio (set status back to Draft).
   This removes it from the Fusion Service workflow immediately.
2. **Re-activate the previous version**: If you previously copied a template
   and kept the original, re-publish the prior version to restore service.
   If not, restore from your last-known-good export (see next point).
3. **Export agent configurations** before any significant change:
   - Use the AI Agent Studio UI to note the exact prompt, topic instructions,
     and tool settings of the stable version (Oracle does not yet provide a
     one-click export/import mechanism — document manually or via screenshot).
4. **Remove from Fusion Service workflows**: If the agent was linked via an
   Action Type, navigate to **Service Center Administration > Action Types**
   and disable or delete the action type to stop it being invoked.
5. **Root-cause investigation**: After rollback, use Agent Tracing to replay
   the failing interactions against the new agent in a test environment before
   re-promoting to production.

### A2A Tool not Discovering Remote Agent

1. Verify the remote agent's base URL is reachable from your OCI region
2. Check that the Agent Card (`/.well-known/agent.json`) is accessible and valid JSON
3. Confirm cross-tenancy IAM policies are in place if the target agent is in
   a different OCI tenancy (see `references/cross-tenancy-policies.md`)
4. Review Agent Tracing for connection errors or JSON parse failures

---


## Examples

### Example 1: Expense Policy FAQ Agent

Create an agent that answers employee questions about corporate expense policies.

**Tool**: Document tool with uploaded expense policy PDF
**Agent prompt**: "You are an expense policy expert. Answer employee questions
about corporate expense policies using the uploaded policy documents. If you
don't know the answer, say so — do not make up policies."
**Topic**: "Expense Policies" with instructions to cover travel, meals, equipment
**Agent Team**: Single-agent team, conversational type

### Example 2: Invoice Processing Multi-Agent Workflow

Automate invoice processing with multiple specialized agents.

**Agents**:
1. *Intake Agent* — Extracts data from incoming invoices (uses Document tool)
   - Prompt: "You are an invoice data extraction specialist. When a user submits
     an invoice document, extract the following fields: vendor name, invoice number,
     invoice date, line items with quantities and unit prices, total amount, currency,
     and payment terms. Output the extracted data in a structured format. If any field
     is unclear or missing, flag it for human review rather than guessing."
2. *Matching Agent* — Performs 3-way matching against POs and receipts
   (uses Business Object tool)
   - Prompt: "You are a procurement matching specialist. Given extracted invoice data,
     query the Fusion Purchasing module to find the corresponding Purchase Order and
     receiving records. Perform 3-way matching: verify that the invoice line items
     match the PO quantities, unit prices (within tolerance), and received quantities.
     Report any discrepancies with specific details."
3. *Validation Agent* — Checks tax rules, policy compliance, duplicate detection
   - Prompt: "You are an AP compliance agent. Check the matched invoice against
     corporate policies: verify tax calculations match jurisdictional rules, check
     for duplicate invoices (same vendor + invoice number + amount), validate
     approval thresholds, and flag any policy violations."
4. *Approval Router* — Routes to appropriate approver based on amount and category

**Agent Team**: Workflow type with chaining. Intake → Matching → Validation →
Human-in-the-loop approval gate → Approval Router.

### Example 3: HCM Skill Recommendation Agent

Deploy the pre-built Skill Recommendation Agent with generative AI:

**Configuration steps**:
1. Stop scheduled process "Synchronize Talent Data for AI Recommendations"
2. Set `ORA_WLF_AI_COMPANY_INFO` at Site level — example value:
   "Accenture is a global professional services company specializing in IT services,
   consulting, and digital transformation. We employ over 700,000 people across 120+
   countries. Our key capabilities include AI strategy, cloud migration, cybersecurity,
   and enterprise application management."
3. Run "Create Summaries Using Generative AI" (Run Type: Full, Object Type: Skills)
4. In AI Agent Studio, search for `SKILL_RECOMMENDATION_AGENT`, click Copy Template
5. In the Agent Designer, review the agent's prompt — customize if needed:
   "You are a skill recommendation assistant for our organization. Based on an
   employee's current role, experience, career goals, and our organization's skill
   taxonomy, recommend relevant skills they should develop. Provide reasoning for
   each recommendation and suggest specific learning resources when available.
   Consider both technical and soft skills."
6. Publish the agent
7. Copy the agent team code from the Agent Teams tab
8. Set `ORA_HRT_SKILL_SUGGESTIONS_AGENT_CODE` to the copied code
9. Set `ORA_HRT_SKILL_SUGGESTIONS` to "Next Generation with Workflow Agents"
10. Schedule "Create Summaries Using Generative AI" to run periodically
    (Run Type: Incremental, Object Type: Skills)

---

## Additional Resources

- Oracle AI Agent Studio documentation: `references/official-docs-links.md`
- Pre-built agents catalog by business function: `references/prebuilt-agents-catalog.md`
- Security considerations (prompt injection, credentials, GDPR, IAM): `references/security-considerations.md`
- OCI AI Database Private Agent Factory (separate product): `references/oci-agent-factory.md`
- Cross-tenancy IAM policies: `references/cross-tenancy-policies.md`
- Oracle AI Agent Studio training and certification: https://mylearn.oracle.com/ou/learning-path/oracle-fusion-ai-agent-studio-foundations-associate/151552

---
> Source: [Suchert/oracle-ai-agent-studio-skill](https://github.com/Suchert/oracle-ai-agent-studio-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
