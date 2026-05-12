## kasa

> Handles manifest file storage with git integration. Files are stored as `<baseDir>/<context>/<namespace>/<app>/<type>.yaml`. The context is the Kubernetes cluster context name, resolved from kubeconfig at startup. The git repo lives at `baseDir`; file operations are scoped to the context subdirectory.

# Kasa - Kubernetes Deployment Agent

## Project Overview

Kasa is a conversational Kubernetes deployment assistant built with Go, using Google's ADK (Agent Development Kit) for LLM agent capabilities and client-go for Kubernetes interaction.

## Verify build

```bash
go build -o /dev/null
```

## Versioning

Use the `bump` CLI tool to manage versions. It updates `.version`, commits, and tags in one step.

```bash
bump -patch   # v0.3.0 → v0.3.1
bump -minor   # v0.3.0 → v0.4.0
bump -major   # v0.3.0 → v1.0.0
```

**Important:** Run `bump` **after** committing your code changes, not before. The bump creates its own commit and tag. If you bump first and then commit code, the tag will point to the version-bump commit instead of the actual changes. If that happens, delete and recreate the tag on the correct commit.

## Run non-interactively:
```
go run . -prompt "list namespaces" # Single prompt mode
go run . -debug -prompt "..."      # With debug output
```

## Configuration

- `~/.kasa/config.yaml` - All settings: Kubernetes, model, API keys, system prompt
- Run `kasa init` to create a default config file
- Environment variables (`OPENROUTER_API_KEY` or `GOOGLE_API_KEY`, `JINA_API_KEY`) override config file values
- Uses OpenRouter (OpenAI-compatible API) by default; any OpenAI-compatible provider works by setting `base_url`

## Project Structure

```
kasa/
├── main.go              # Entry point, agent setup
├── openaimodel/         # Vendored OpenAI-compatible model adapter for ADK
├── tools/               # All K8s tools (one file per tool, see tools.go for registry)
├── repl/                # Interactive REPL with plan/approval workflow
├── manifest/            # Manifest file storage with git integration
├── references/          # Embedded K8s resource documentation
└── deployments/         # Git-tracked manifest storage (created at runtime)
```

## Safe Mode (Plan/Approval Workflow)

Kasa operates in **Safe Mode** by default in interactive REPL. Mutating operations require user approval before execution.

### How It Works

1. User requests a change (e.g., "deploy nginx")
2. Agent gathers information using read-only tools
3. Agent calls `propose_plan` with description and actions
4. REPL detects the plan and displays it for review
5. User types `/approve` to approve or `/abort` to reject
6. If approved, agent executes the planned actions

### Tool Categories

Tools are classified in `tools/tools.go`:

**Read-Only (use freely):**
- list_namespaces, list_pods, get_logs, get_events, get_resource
- get_reference, check_deployment_health
- list_manifests, read_manifest, dry_run_apply (supports inline YAML validation)
- list_resources (generic, supports CRDs)

**Mutating (require plan approval):**
- delete_namespace, delete_resource, delete_manifest
- apply_resource (single tool for all resource creation/updates via YAML)
- apply_manifest, import_resource, commit_manifests

### REPL Commands

- `/approve` - Approve pending plan
- `/abort` (or `/reject`) - Reject pending plan
- `/plan` - Display pending plan again
- `/debug` - Toggle debug mode

### Key Files

- `session_state.go` - `SessionState`, `Plan`, `PlannedAction` types
- `plan_display.go` - `DisplayPlan()`, `ParsePlanFromResponse()`, `FormatExecutionPrompt()`
- `tools/propose_plan.go` - The `propose_plan` tool
- `tools/tools.go` - `IsMutating()`, `ReadOnlyTools()`, `MutatingTools()`

### Non-Interactive Mode

When using `-prompt`, safe mode is disabled (no approval workflow). The agent executes directly.

## Key Patterns

### Tool Implementation

Tools implement the `tool.Tool` interface from `google.golang.org/adk/tool`. Each tool is a struct with these methods:

```go
type MyTool struct {
    // dependencies (e.g., clientset, config)
}

func NewMyTool(...) *MyTool {
    return &MyTool{...}
}

func (t *MyTool) Name() string {
    return "my_tool"
}

func (t *MyTool) Description() string {
    return "What this tool does"
}

func (t *MyTool) IsLongRunning() bool {
    return false
}

func (t *MyTool) ProcessRequest(ctx tool.Context, req *model.LLMRequest) error {
    return addFunctionTool(req, t)
}

func (t *MyTool) Declaration() *genai.FunctionDeclaration {
    return &genai.FunctionDeclaration{
        Name:        t.Name(),
        Description: t.Description(),
        Parameters: &genai.Schema{
            Type: "object",  // NOTE: string, not genai.TypeObject
            Properties: map[string]*genai.Schema{
                "param_name": {
                    Type:        "string",
                    Description: "Parameter description",
                },
            },
            Required: []string{"param_name"},
        },
    }
}

func (t *MyTool) Run(ctx tool.Context, args any) (map[string]any, error) {
    argsMap, ok := args.(map[string]any)
    if !ok {
        return map[string]any{"error": "invalid arguments"}, nil
    }

    // Execute tool logic
    result := doSomething(argsMap["param_name"].(string))

    return map[string]any{
        "result": result,
    }, nil
}
```

### Adding a New Tool

1. Create `tools/mytool.go` with the struct and all interface methods
2. Register in `tools/tools.go` by adding to `All()`:
   ```go
   func (k *KubeTools) All() []tool.Tool {
       return []tool.Tool{
           // ... existing tools ...
           NewMyTool(k.clientset),                    // K8s typed client only
           NewMyTool(k.dynamicClient),                // Dynamic client only (for CRDs)
           NewMyTool(k.clientset, k.dynamicClient),   // Both clients
           NewMyTool(k.manifest),                     // Manifest-only tool
           NewMyTool(k.clientset, k.dynamicClient, k.manifest), // All three
       }
   }
   ```
3. If the tool modifies state, add it to `mutatingToolNames` in `tools/tools.go`:
   ```go
   var mutatingToolNames = map[string]bool{
       // ... existing tools ...
       "my_tool": true,
   }
   ```
4. Build and test

### Agent Architecture

The agent uses ADK's runner/session pattern with an OpenAI-compatible model adapter:

```go
// Create LLM model via OpenAI-compatible API (OpenRouter, etc.)
llmModel := openaimodel.New(modelName, apiKey, baseURL, retryTransport)

// Create agent with tools
agent, _ := llmagent.New(llmagent.Config{
    Name:        "kasa",
    Model:       llmModel,
    Instruction: systemPrompt,
    Tools:       kubeTools.All(),
})

// Run with session
sessionService := session.InMemoryService()
r, _ := runner.New(runner.Config{
    AppName:        "kasa",
    Agent:          agent,
    SessionService: sessionService,
})

// Execute and stream response
for event, err := range r.Run(ctx, userID, sessionID, userMessage, agent.RunConfig{}) {
    // Handle events
}
```

### References Package

Embedded Kubernetes resource documentation in `references/data/*.md`. Access via:

```go
references.List()                    // []string of available topics
references.ListWithDescriptions()    // map[string]string with descriptions
references.Lookup("deployment")      // Returns markdown content
```

### Dynamic Client & CRD Support

Kasa uses both typed clients (for core resources) and a dynamic client (for CRDs and unknown resources). The `tools/gvr.go` file contains:

- `CommonGVRs` - Map of 30+ known resource kinds to their GroupVersionResource
- `KindAliases` - User-friendly aliases (e.g., `deploy` → `deployment`, `gw` → `gateway`)
- Helper functions: `NormalizeKindName()`, `LookupGVR()`, `IsNamespaced()`, `ParseYAMLToUnstructured()`, `GVKToGVR()`

**Supported CRDs out of the box:**
- **Gateway API**: Gateway, HTTPRoute, GRPCRoute, TCPRoute, UDPRoute, TLSRoute, ReferenceGrant, GatewayClass
- **cert-manager**: Certificate, Issuer, ClusterIssuer, CertificateRequest
- **Autoscaling**: HorizontalPodAutoscaler

**Generic tools for any resource:**
- `apply_resource` - Apply any YAML manifest (creates or updates)
- `list_resources` - List any resource type by kind
- `get_resource` - Get any resource (falls back to dynamic client for unknown kinds)
- `import_resource` - Import any resource from cluster to manifests
- `delete_resource` - Delete any resource type

For unknown CRDs, provide the `api_version` parameter (e.g., `gateway.networking.k8s.io/v1`).

### Manifest Package

Handles manifest file storage with git integration. Files are stored as `<baseDir>/<context>/<namespace>/<app>/<type>.yaml`. The context is the Kubernetes cluster context name, resolved from kubeconfig at startup. The git repo lives at `baseDir`; file operations are scoped to the context subdirectory.

```go
manager, _ := manifest.NewManager("~/deployments", "my-cluster-context")
manager.EnsureGitInit()

// Save and stage a manifest
path, _ := manager.SaveManifest("default", "nginx", "deployment", yamlBytes)

// List manifests (filter by namespace/app, empty = all)
manifests, _ := manager.ListManifests("default", "")  // []ManifestInfo

// Read manifest content
content, _ := manager.ReadManifest("default", "nginx", "deployment")

// Delete manifest(s) and stage deletion (empty type = delete all for app)
deleted, _ := manager.DeleteManifest("default", "nginx", "deployment")

// Commit staged changes
manager.Commit("Deploy nginx to default namespace")
```

## Future: ADK Tool Confirmation

ADK v0.4.0 added built-in Human-in-the-Loop tool confirmation (`tool/toolconfirmation` package). This overlaps with kasa's custom plan/approval workflow but operates at a **per-tool-call** level rather than batch plan approval. Three modes:

1. `functiontool.Config{RequireConfirmation: true}` — always confirm before execution
2. `RequireConfirmationProvider: func(args T) bool` — conditional confirmation
3. Manual `ctx.ToolConfirmation()` / `ctx.RequestConfirmation()` — full control inside `Run()`

The wire protocol uses a special `adk_request_confirmation` FunctionCall event; the client responds with a FunctionResponse containing `{"confirmed": bool, "payload": ...}`.

Kasa currently uses a higher-level abstraction (batch plan approval via `propose_plan` tool + `/approve` REPL command). Migrating to ADK's built-in confirmation could simplify the implementation but would change the UX from "approve a plan" to "approve each tool call individually". Consider whether a hybrid approach makes sense.

See `examples/toolconfirmation/main.go` in the ADK repo for a full Go example.

## Dependencies

- `google.golang.org/adk` - Agent Development Kit (runner, session, tool interfaces)
- `google.golang.org/genai` - Gemini/genai types (used by ADK tool interface)
- `github.com/sashabaranov/go-openai` - OpenAI API client (used by openaimodel adapter and REPL commit messages)
- `k8s.io/client-go` - Kubernetes typed client and dynamic client
- `k8s.io/apimachinery` - Kubernetes API types and unstructured objects
- `gopkg.in/yaml.v3` - Config parsing
- `sigs.k8s.io/yaml` - YAML/JSON conversion for Kubernetes objects

## Analyzing Session Dumps

Kasa dumps session event logs to `~/.kasa/dump-<timestamp>.json` when the agent hits the tool call hard limit. These are useful for diagnosing model behavior (loops, warning compliance, token efficiency).

Dumps are large JSON files. Use this Python one-liner to get a summary:

```bash
python3 -c "
import json, sys
with open(sys.argv[1]) as f:
    data = json.load(f)
print(f'Total events: {data[\"event_count\"]}')
tp, tc, tt = 0, 0, 0
for i, ev in enumerate(data['events']):
    parts = ev.get('Content', {}).get('parts', [])
    role = ev['Content'].get('role', '?')
    desc = []
    for p in parts:
        if 'text' in p: desc.append(f'text: {p[\"text\"][:100].replace(chr(10), \" \")}')
        if 'functionCall' in p: desc.append(f'call: {p[\"functionCall\"][\"name\"]}')
        if 'functionResponse' in p:
            r = json.dumps(p['functionResponse'].get('response', {}), default=str)[:100]
            desc.append(f'resp: {p[\"functionResponse\"][\"name\"]} -> {r}')
    usage = ev.get('UsageMetadata')
    tokens = ''
    if usage:
        p_, c, t = usage.get('promptTokenCount',0), usage.get('candidatesTokenCount',0), usage.get('thoughtsTokenCount',0)
        tp += p_; tc += c; tt += t
        tokens = f' [p:{p_} c:{c} t:{t}]'
    print(f'{i:3d} [{role}]{tokens} {\" | \".join(desc)}')
print(f'\nTotals: prompt={tp} candidate={tc} thought={tt}')
" ~/.kasa/dump-NNNN.json
```

Key things to look for:
- **Total events vs tool calls**: high ratio = model is looping
- **`_warning` in responses**: check if the model pivoted or ignored them
- **Repeated tool names with similar args**: the hallmark of a stuck loop
- **Whether a final text answer was given**: did the model complete or get cut off?

See `PATHOLOGICAL.md` for a benchmark comparing model behavior on a known loop-inducing prompt.

## Testing

Requires:
- Valid kubeconfig at `~/.kube/config`
- `OPENROUTER_API_KEY` (or `GOOGLE_API_KEY`) set in `~/.kasa/config.yaml` or as environment variable
- `agent.model` set in config (e.g., `anthropic/claude-sonnet-4`, `google/gemini-2.5-flash`)

---
> Source: [perbu/kasa](https://github.com/perbu/kasa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
