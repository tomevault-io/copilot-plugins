## codejam-code-based-agents

> description: "Use when writing Python code for SAP AI Core integrations: CrewAI agents, LiteLLM models, SAP-RPT-1 API calls, Grounding Service queries, OAuth2 token handling. Covers tool patterns, model string format, error handling, and YAML/decorator conventions."

---
description: "Use when writing Python code for SAP AI Core integrations: CrewAI agents, LiteLLM models, SAP-RPT-1 API calls, Grounding Service queries, OAuth2 token handling. Covers tool patterns, model string format, error handling, and YAML/decorator conventions."
applyTo: "**/*.py"
---

# Python SAP Tools Conventions

## LLM Model Strings

Use the `sap/<model-name>` format matching the deployment name in SAP AI Launchpad:

```python
llm = LLM(model="sap/gpt-4o")
```

Never use bare provider strings like `"gpt-4o"` or `"openai/gpt-4o"` for SAP AI Core deployments.

## CrewAI Agent & Task Pattern

Agents and their tasks are defined in YAML; the Python method names **must exactly match** the YAML keys:

```python
@CrewBase
class MyCrew():
    agents_config = "config/agents.yaml"
    tasks_config  = "config/tasks.yaml"

    @agent
    def appraiser_agent(self) -> Agent:   # key in agents.yaml: appraiser_agent
        return Agent(config=self.agents_config["appraiser_agent"], tools=[call_rpt1])

    @task
    def appraisal_task(self) -> Task:     # key in tasks.yaml: appraisal_task
        return Task(config=self.tasks_config["appraisal_task"])

    @crew
    def crew(self) -> Crew:
        return Crew(agents=self.agents, tasks=self.tasks, process=Process.sequential)
```

Always use `Process.sequential`; task outputs are automatically passed as context to subsequent tasks.

## Tool Pattern

Tools are module-level functions decorated with `@tool`. Return error messages as plain strings so the LLM can recover gracefully — never raise exceptions out of a tool:

```python
from crewai.tools import tool

@tool("call_rpt1")
def call_rpt1(payload: dict) -> str:
    """Docstring is the tool description shown to the agent."""
    try:
        response = rpt1_client.post_request(json_payload=payload)
        if response.status_code == 200:
            return json.dumps(response.json(), indent=2)
        return f"Error {response.status_code}: {response.text}"
    except Exception as e:
        return f"Error calling RPT-1: {str(e)}"
```

## OAuth2 Client-Credentials Pattern (SAP AI Core)

```python
import os, requests

data = {
    "grant_type": "client_credentials",
    "client_id": os.getenv("AICORE_CLIENT_ID"),
    "client_secret": os.getenv("AICORE_CLIENT_SECRET"),
}
headers = {"Content-Type": "application/x-www-form-urlencoded"}
resp = requests.post(os.getenv("AICORE_AUTH_URL"), data=data, headers=headers, timeout=30)
resp.raise_for_status()
token = resp.json()["access_token"]
```

Subsequent requests use `"Authorization": f"Bearer {token}"` and `"AI-Resource-Group": resource_group`.

> **Token expiry**: the token is fetched once at init. For long-running crews, re-instantiate the client to refresh.

## Grounding Service Query Pattern

```python
from gen_ai_hub.document_grounding.client import RetrievalAPIClient
from gen_ai_hub.document_grounding.models.retrieval import RetrievalSearchInput, RetrievalSearchFilter
from gen_ai_hub.orchestration.models.document_grounding import DataRepositoryType

@tool("call_grounding_service")
def call_grounding_service(user_question: str) -> str:
    """Query evidence documents via SAP Grounding Service."""
    client = RetrievalAPIClient()
    search_filter = RetrievalSearchFilter(
        id="vector",
        dataRepositoryType=DataRepositoryType.VECTOR.value,
        dataRepositories=["<YOUR_PIPELINE_ID>"],  # Replace with pipeline ID from SAP AI Launchpad
        searchConfiguration={"maxChunkCount": 5},
    )
    search_input = RetrievalSearchInput(query=user_question, filters=[search_filter])
    response = client.search(search_input)
    return json.dumps(response.model_dump(), indent=2)
```

Replace `<YOUR_PIPELINE_ID>` with the vector DB pipeline ID from SAP AI Launchpad before running.

## RPT-1 Payload

Use the string `"[PREDICT]"` as the placeholder for values the model should infer. The schema (dtype, categories, value ranges) must match the deployed model exactly. Pass the payload dict directly from `main.py` inputs via the crew kickoff:

```python
crew.kickoff(inputs={"payload": payload, "query": "Who stole the art?"})
```

## Environment Variables

Load `.env` once at the top of the entry-point module, before instantiating any client:

```python
from pathlib import Path
from dotenv import load_dotenv

load_dotenv(dotenv_path=Path(__file__).parent / ".env")
```

Required keys: `AICORE_CLIENT_ID`, `AICORE_CLIENT_SECRET`, `AICORE_AUTH_URL`, `RPT1_DEPLOYMENT_URL`, `AICORE_RESOURCE_GROUP`, `AICORE_BASE_URL`.

---
> Source: [SAP-samples/codejam-code-based-agents](https://github.com/SAP-samples/codejam-code-based-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
