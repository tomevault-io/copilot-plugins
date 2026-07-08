## agentgateway-demos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A collection of self-contained **AgentGateway** demos and workshops. AgentGateway (https://agentgateway.dev) is an open-source proxy for LLM and MCP traffic. Each numbered top-level directory (`01-…` through `10-…`) is an independent demo with its own scripts, config, and readme — there is no shared build system, package manifest, or test runner across the repo. Work inside one demo directory at a time.

## Two deployment modes

Demos use one of two ways to run AgentGateway. Identify which a demo uses before editing.

1. **Kubernetes** (`01`, `03`, `04`, `05`, `06`, `07`, `09`) — A local `kind` cluster + Helm install of the AgentGateway control plane, configured via Gateway API resources. The LLM/MCP provider is a custom `AgentgatewayBackend` CRD (`agentgateway.dev/v1alpha1`); traffic is routed to it with a standard `HTTPRoute` whose `backendRefs` point at the backend with `group: agentgateway.dev, kind: AgentgatewayBackend`. API keys are stored in Kubernetes `Secret`s.

2. **Standalone** (`00`, `08`) — The `agentgateway` binary run directly against a `config.yaml`: `agentgateway -f config.yaml`. The standalone config schema is different from the K8s CRDs — it nests `binds → listeners → routes → backends`. Proxy listener is `:3000`, admin UI is `http://localhost:15000/ui/`. Validate config against `# yaml-language-server: $schema=https://agentgateway.dev/schema/config`.

## Per-demo conventions

K8s demos share a consistent script set — match it when adding a demo:

- `deploy.sh` — preflight-checks tools (`kind`, `kubectl`, `helm`, `jq`) and required API-key env vars, creates the kind cluster (idempotent: skips if it exists), installs Gateway API CRDs + AgentGateway Helm charts, then applies Gateway/Backend/HTTPRoute resources.
- `test.sh` — port-forwards `svc/agentgateway-proxy` and sends `curl` requests to verify routing/behavior.
- `cleanup.sh` — deletes resources and the kind cluster.
- `step-by-step.sh` — annotated walkthrough version of the deploy (echoes each command as it runs); used for live demos.
- `readme.md` / `README.md` — architecture diagram, key concepts, quick start, manual steps.

Each deploy script pins its own versions and **its own cluster name** (clusters do not collide, so demos can run simultaneously):

| Demo | Cluster name | AGW version |
|------|--------------|-------------|
| 01-installagentgateway | `agw-install` | v1.1.0 |
| 03-loadbalancing-models | `agw-loadbalancing` | v1.0.1 |
| 04-vitural-keys | `agw-virtual-keys` | v1.1.0 |
| 05-content-based | `agw-content-based` | v1.1.0 |
| 06-virtual-mcp | `agw-series-demo` | v1.1.0 |
| 09-k8s-langfuse | `agw-k8s-langfuse` | v1.1.0 |
| 103-agw-tokenomics-with-f5-tool-modes | `agw-f5-tool-modes` | v2026.6.1 |
| 104-ent-github-tokenomics | `agw-github-tokenomics` | v2026.6.1 |
| 105-ent-headroom-comp-tokenomics | `agw-headroom-comp` | v2026.6.1 |

Demos `103`, `104`, and `105` use the **Enterprise** AgentGateway (`EnterpriseAgentgatewayBackend`,
`entMcp.toolMode` Standard/Search/Code) from `oci://us-docker.pkg.dev/solo-public/enterprise-agentgateway/charts/`, not the OSS charts above. `104` and `105` front an **external** MCP server (GitHub's hosted `api.githubcopilot.com/mcp`) — no in-cluster MCP pod. `105` forks `104` and adds a second knob: a local **Headroom** compression proxy (https://github.com/headroomlabs-ai/headroom) the harness routes the LLM call through (`HEADROOM=on` + `LLM_URL`), to test whether AGW's catalog savings and Headroom's payload compression *stack*. Headroom defaults to compression OFF — `105`'s `run_matrix.sh`/`test.sh` launch it with compression explicitly enabled.

Gateway API CRDs are `v1.5.0` everywhere. Namespace is `agentgateway-system`. Helm charts come from `oci://cr.agentgateway.dev/charts/` (`agentgateway-crds` + `agentgateway`).

## Running a K8s demo

```bash
cd 03-loadbalancing-models          # pick a demo
export OPENAI_API_KEY="..."         # set the keys that demo's deploy.sh checks for
export ANTHROPIC_API_KEY="..."      # (load-balancing demos need both)
./deploy.sh
kubectl port-forward -n agentgateway-system svc/agentgateway-proxy 8080:80 &
./test.sh
./cleanup.sh                        # tears down the kind cluster
```

## Secrets

API keys are passed as **environment variables**, never committed. `deploy.sh` reads them from the env and creates K8s Secrets; standalone demos use `${VAR}` placeholders in `config.yaml` that `run.sh` substitutes at runtime (via `envsubst`, falling back to `sed`) into a temp file so the real secret never lands in the tracked config. The root `.env` and per-demo `.env` files are gitignored — keep real keys there, commit `.env.example` instead.

## Langfuse observability (08, 09)

AgentGateway emits OpenTelemetry traces using GenAI semantic conventions natively (no app code changes). These are pointed at a Langfuse OTLP endpoint (`.../api/public/otel/v1/traces`) via the `config.tracing` block (standalone) or equivalent. Auth is a base64 `public_key:secret_key` string in `LANGFUSE_AUTH_STRING`. Force `otlpProtocol: http` — Langfuse ingest does not support gRPC.

---
> Source: [sebbycorp/agentgateway-demos](https://github.com/sebbycorp/agentgateway-demos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
