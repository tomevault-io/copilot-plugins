## theta-edgecloud-hermes-skill

> Use Theta EdgeCloud from Hermes for cost-conscious LLM, on-demand media/inference, dedicated OpenAI-compatible endpoints, GPU/deployment checks, and Theta Video workflows with command-scoped credentials and dry-run safety.


# Theta EdgeCloud for Hermes

Use this skill when the user wants Hermes to reduce AI execution costs or route suitable model, media, inference, video, and GPU workloads through Theta EdgeCloud.

This Hermes skill is adapted from the OpenClaw Theta EdgeCloud skill by Zeus Labs / Theta Communications. It is designed as a public, inspectable Hermes skill that other users can install and use without exposing credentials.

## Credits and support

Credit: Theta Communications (`thetacommunications.com`). If you want to support more projects like this, stake THETA/TFUEL with Theta Communications or donate at `https://www.thetacommunications.com/donations`.

## Safety and credential model

Credentials are command-scoped. Do **not** ask users to configure every variable unless they need every command family.

- Controller/deployment/project commands:
  - `THETA_EC_API_KEY`
  - `THETA_EC_PROJECT_ID`
- Balance command:
  - `THETA_ORG_ID`
- On-demand inference/media commands:
  - `THETA_ONDEMAND_API_TOKEN` or `THETA_ONDEMAND_API_KEY` or `THETA_API_KEY`
  - Theta's official MCP server expects this same token value under `THETA_API_KEY`.
- Dedicated OpenAI-compatible inference endpoint:
  - `THETA_INFERENCE_ENDPOINT`
  - either `THETA_INFERENCE_AUTH_TOKEN`
  - or `THETA_INFERENCE_AUTH_USER` + `THETA_INFERENCE_AUTH_PASS`
- Theta Video API:
  - `THETA_VIDEO_SA_ID`
  - `THETA_VIDEO_SA_SECRET`

Security rules:

1. Never print or log raw Theta credentials.
2. Prefer environment variables or Hermes profile `.env` entries; never put secrets in git-tracked files.
3. For mutating or paid operations, confirm user intent unless `THETA_DRY_RUN=1` is set.
4. Dedicated inference endpoint override via ad-hoc args should be avoided; prefer `THETA_INFERENCE_ENDPOINT`.
5. Treat `404`, `502`, `503`, and `504` shortly after endpoint creation as possible warm-up, not immediate permanent failure.

## Quick setup for users

1. Log in at `https://www.thetaedgecloud.com/`.
2. Go to **Account -> Projects** and select your project.
3. Click **Create API Key** and copy the API key and project id.
4. Add only the env vars needed for your workflow.
5. For dedicated GPU deployments, check **Account -> Quota** and request/increase quota if needed.
6. For safest first run, set `THETA_DRY_RUN=1` and validate list/read endpoints before create/delete operations.

## Hermes helper script

This skill includes `scripts/theta_edgecloud.py`, a dependency-free Python helper. Run it from the skill directory or after installing the skill:

```bash
python scripts/theta_edgecloud.py setup
python scripts/theta_edgecloud.py capabilities
python scripts/theta_edgecloud.py ondemand-list-services
python scripts/theta_edgecloud.py ondemand-chat --service qwen3 --message "Say hello"
python scripts/theta_edgecloud.py ondemand-chat --service gpt_oss_120b --message "Say hello"
python scripts/theta_edgecloud.py ondemand-infer --service stable_diffusion_xl_turbo --prediction predict --payload-json '{"input":{"prompt":"blue edge-cloud icon","steps":2,"strength":0.7,"guidance":0}}' --poll
python scripts/theta_edgecloud.py controller-vm-types
python scripts/theta_edgecloud.py controller-balance
python scripts/theta_edgecloud.py controller-standard-templates --category serving
python scripts/theta_edgecloud.py controller-list-deployments
python scripts/theta_edgecloud.py controller-lifecycle-deployment --action stop --deployment-id base_demo --project-id prj_demo --dry-run
python scripts/theta_edgecloud.py controller-validate-disposable --dry-run --org-id org_demo --probe openai --payload-json '{"project_id":"prj_demo","deployment_template_id":"img_demo"}'
python scripts/theta_edgecloud.py ondemand-upload-url --service whisper --input-field audio_filename --dry-run
python scripts/theta_edgecloud.py dedicated-ready --probe openai
python scripts/theta_edgecloud.py dedicated-models
python scripts/theta_edgecloud.py dedicated-chat --message "Say hello"
```

Use `--json` where available for machine-readable output.

## Official Theta MCP server

Theta Labs publishes an official MCP server for Theta EdgeCloud On-Demand Model APIs: `@thetalabs/on-demand-api-mcp`. Prefer this official MCP server for generic on-demand model access where it is validated instead of duplicating those tools here.

Hermes config example:

```yaml
mcp_servers:
  theta_ondemand:
    command: "npx"
    args: ["@thetalabs/on-demand-api-mcp"]
    env:
      THETA_API_KEY: "REPLACE_WITH_THETA_ONDEMAND_ACCESS_TOKEN"
    timeout: 180
    connect_timeout: 60
    sampling:
      enabled: false
```

After adding config, restart Hermes or run `/reload-mcp`. Expected Hermes tool names use the configured server prefix, for example `mcp_theta_ondemand_list_services`, `mcp_theta_ondemand_infer`, `mcp_theta_ondemand_get_request_status`, and `mcp_theta_ondemand_get_upload_url`.

For `gpt_oss_120b` through the official MCP `infer` tool, include `stream: false` inside `input` to avoid SSE/JSON parse errors:

```json
{
  "service": "gpt_oss_120b",
  "input": {
    "messages": [{"role": "user", "content": "Say hello"}],
    "max_tokens": 64,
    "temperature": 0.3,
    "stream": false
  },
  "wait": 60
}
```

This skill remains useful for Hermes-specific setup guidance, safety practices, controller/project APIs, balance checks, dedicated OpenAI-compatible endpoints, and disposable deployment validation.

Supporting files:

- `references/hermes-theta-official-mcp-config.yaml` — copy/paste Hermes config example.
- `references/official-theta-mcp-hermes-setup.md` — full setup and troubleshooting notes.
- `references/structured-examples.md` — copy/paste examples for `gpt_oss_120b`, `qwen3`, image/audio/video, LLaVA, and dedicated validators.
- `scripts/validate_official_mcp.sh` — checks Node/npm/package metadata.
- `scripts/test_official_mcp_hermes.sh` — verifies Hermes can discover the 4 official MCP tools in a temporary profile.

## On-demand service guidance

Live validation on 2026-06-09 confirmed:

- `ondemand-list-services` returned 6 live services: `gpt_oss_120b`, `blip`, `qwen3`, `whisper`, `stable_diffusion_xl_turbo`, `llava`.
- `gpt_oss_120b` chat succeeded through `/infer_request/chat/completions` with OpenAI-compatible JSON, including `choices`, `usage`, and a `reasoning` field in the message object. Retest latency was about `1.128s` for a short prompt; usage was `75` prompt tokens, `76` completion tokens, `151` total tokens.
- `stable_diffusion_xl_turbo` image generation succeeded through generic `ondemand-infer`; a 2-step test returned an `image_url` and reported `cost_usd.output = 0.01`.
- `qwen3` returned `409 No instances available - try again later` during multiple live tests, so treat it as capacity-sensitive and retry later rather than marking credentials invalid.

Validated aliases from the OpenClaw skill include:

- Chat/LLM: `qwen3`, `gpt_oss_120b`, `llama_3_1_70b`
- Image/vision/audio: `flux`, `stable_diffusion_xl_turbo`, `grounding_dino`, `blip`, `llava`, `whisper`
- Catalog-only/stale in the source skill: `minimax_m2_5`, `llama_3_8b`, `step_video`, `esrgan`, `voice_cloning`, `instant_id`, `talking_head`

Qwen3 notes:

- Canonical slug: `qwen3`
- Request family: chat/completions
- Payload shape: `input.messages = [{ role, content }]`
- Streaming can take 30-50s before returning text.
- Capacity can temporarily return `409 No instances available`; retry later.

GPT OSS 120B notes:

- Canonical slug: `gpt_oss_120b`
- Use OpenAI-compatible `POST /infer_request/chat/completions` with `model: "gpt_oss_120b"`.
- Do not route GPT OSS through the generic `/infer_request/gpt_oss_120b?prediction=completions` wrapper; the source skill observed null output on that path.

## Dedicated inference guidance

Dedicated OpenAI-compatible inference (`/v1/models`, `/v1/chat/completions`) can work after project quota / Developer Plan readiness.

Warm-up behavior observed in the source skill:

- authenticated `GET /v1/models` can first return transient `404`
- then transient `502`
- then success
- authenticated `POST /v1/chat/completions` succeeds after warm-up

Operational rule: retry authenticated readiness for 1-2 minutes before declaring dedicated endpoint failure.

Dedicated endpoint credentials are not the same as the project API key. The project API key can manage controller/project resources when it has permission, including creating a disposable dedicated serving deployment. A successful disposable-deployment test path can generate temporary Basic Auth credentials (`auth_username` / `auth_password`) in the deployment create payload, then read the resulting endpoint URL and auth fields from the create/list response, call `/v1/models` and `/v1/chat/completions` with Basic Auth, and delete the deployment afterward.

Correct operational model:

- `THETA_INFERENCE_ENDPOINT` can be discovered from a live dedicated deployment create/list response, but it cannot be derived from only the project API key without an actual deployment.
- `THETA_INFERENCE_AUTH_TOKEN` is optional and was not required for the successful OpenClaw validation path.
- The successful disposable deployment path used generated `THETA_INFERENCE_AUTH_USER` / `THETA_INFERENCE_AUTH_PASS` style Basic Auth credentials.
- Creating dedicated deployments may spend credits and requires quota/plan readiness. Treat it as a paid mutating operation.

## Disposable deployment live validation (2026-06-10)

Validated project-scoped deployment permissions with a real Theta EdgeCloud Project API key and `THETA_EC_PROJECT_ID` in `prj_...` format. `controller-list-deployments` and `controller-custom-templates` returned `200` after the correct project id was supplied; a previous non-`prj_` value caused `403 You are not allowed to perform this action`.

Live disposable dedicated serving deployment validation used the cheapest suitable standard serving template observed at the time:

- Template: `Grounding Dino`
- Template id: `img_qqu31asazaig666jtzp7gjd4pway`
- Image: `thetalabsofficial/grounding-dino:1.3`
- VM: `vm_gt1` / `G-T4_16GB-x1`
- Container port: `7860`
- Auth: generated `auth_username` / `auth_password` Basic Auth in the create payload

Observed behavior:

- `POST https://controller.thetaedgecloud.com/deployment` returned `200` with `Endpoint`, `BaseID`, `Shard`, `Suffix`, `AuthUsername`, `AuthPassword`, `EndpointStatus`, `Region`, and resource fields.
- Endpoint warm-up sequence included `404` -> repeated `503` while the pod was Pending -> `401` endpoint status while proxy/auth gating came online -> app readiness.
- Authenticated `/config` eventually returned `200` with Gradio config (`version: 3.41.2`).
- `/v1/models` returned `404` for this Gradio/WebUI template; OpenAI-compatible `/v1/*` should only be expected for vLLM/OpenAI-compatible serving templates.
- Real Gradio prediction via `POST /api/predict` succeeded with `fn_index: 0` and input shape `[image, text prompt, box threshold slider, text threshold slider]`, returning a base64 image result.
- Cleanup with `DELETE /deployment/{Shard}/{Suffix}?project_id=...` returned `200`; post-delete deployment list returned `[]`.

Budget note: this validation consumed about `6.666666` Theta EdgeCloud balance units based on pre/post balance readings (`3421.300894` -> `3414.634228`). Future disposable validation should use one run only and stop immediately after `/config` or first prediction success unless a larger budget is explicitly approved.

Additional OpenAI-compatible dedicated validation attempt: `gpt-oss-20b` on recommended `vm_gh200x1` / H200 created successfully and returned endpoint + Basic Auth handles, but did not reach `/v1/models` readiness within a 10-minute polling window. Observed sequence was initial `404`, then repeated `502` for about 4 minutes, then repeated `404` until timeout. `/v1/chat/completions` was not attempted because `/v1/models` never returned `200`. Cleanup succeeded and final deployment list was empty. Balance changed from `3363.134229` to `3321.634229`, delta `41.5` balance units. Treat long H200 warm-up tests as expensive; prefer on-demand `gpt_oss_120b` for cost-conscious chat unless a persistent dedicated endpoint is needed.

Successful OpenAI-compatible dedicated validation: `DeepSeek R1 / Distill-Qwen-7B` (`img_gf070dbq0kttgz1atmiatyunhdxm`, image `thetalabsofficial/vllm-theta:latest`) on recommended `vm_gh200x1` created successfully, reached `/v1/models` readiness after about 655.6 seconds, and `POST /v1/chat/completions` returned `200` in about 1.024 seconds. Model id returned: `deepseek-ai/DeepSeek-R1-Distill-Qwen-7B`. Usage for the short smoke prompt: prompt tokens `14`, completion tokens `64`, total tokens `78`. Status sequence was initial `404`, repeated `503`, repeated `500`, then `200`. Cleanup succeeded with zero remaining deployments. Balance delta was `41.5` units. This proves Theta dedicated can serve an OpenAI-compatible chat endpoint for Hermes, but cold-start time/cost make it better as a persistent endpoint or pre-warmed fallback than per-request spin-up.

## Controller/project API guidance

Official Theta docs expose an AI-agent-friendly documentation index at `https://docs.thetatoken.org/llms.txt`; individual pages can be fetched as Markdown by appending `.md` (for example `https://docs.thetatoken.org/docs/use-edgecloud-api-keys-to-manage-deployments.md`). Use this for future maintenance instead of scraping rendered ReadMe HTML.

Read-only helper commands include:

- `controller-vm-types` — public VM type catalog from `api.thetaedgecloud.com`.
- `controller-balance` — organization balance lookup using `THETA_ORG_ID`.
- `controller-standard-templates --category serving` — standard serving templates from `controller.thetaedgecloud.com`.
- `controller-custom-templates` — project custom templates.
- `controller-list-deployments` — project deployments.
- `controller-create-deployment --payload-json ... --yes` — advanced mutating create wrapper; use `THETA_DRY_RUN=1` or `--dry-run` first.
- `controller-delete-deployment --deployment-id ... --yes` — advanced delete wrapper for cleanup.

Controller APIs are Cloudflare-fronted. The helper sends a browser-like `User-Agent`; Python/urllib's default user-agent can receive Cloudflare `403 Error 1010` even for public catalog endpoints.

If `controller-list-deployments` returns `403 {"status":"error","message":"You are not allowed to perform this action"}`, verify that `THETA_EC_API_KEY` is the actual project API key from **Account -> Projects -> Create API Key** and not just the project id.

For disposable dedicated inference validation, the recommended future helper flow is: list serving templates, choose a low-cost/quota-compatible template and VM, generate random Basic Auth username/password, create deployment with those auth fields, poll/list until endpoint is returned, retry `/v1/models` through warm-up, run one `/v1/chat/completions` request, then delete the deployment.

The helper includes `controller-validate-disposable` for this workflow. It defaults to dry-run-safe behavior through `THETA_DRY_RUN=1` or `--dry-run`, injects generated Basic Auth if missing, redacts auth fields in output, refuses real paid/mutating execution unless `--yes` is passed, polls either `openai` (`/v1/models`) or `gradio` (`/config`) readiness, attempts deletion in cleanup after a real create, verifies cleanup from deployment listing when possible, and reports pre/post org balance plus numeric delta when `--org-id` or `THETA_ORG_ID` is configured.

Additional v0.2 helper coverage:

- `controller-lifecycle-deployment --action start|stop` performs guarded start/stop calls. Use dry-run first; the helper can try singular `/deployment/...` and plural `/deployments/...` route styles because Theta docs/runtime examples have used both.
- `ondemand-upload-url` calls the on-demand presigned upload URL API for file inputs such as Whisper audio.
- `dedicated-ready` provides a dedicated endpoint readiness probe with configurable timeout/interval for OpenAI-style `/v1/models` or Gradio `/config` endpoints.

Official deployment-create payload fields include `project_id`, `deployment_template_id`, `container_image`, `vm_id`, `min_replicas`, `max_replicas`, `env_vars`, `annotations`, `auth_username`, `auth_password`, `registry_username`, `registry_password`, and Jupyter-only `password`. Successful create/list responses include `Endpoint`, `Shard`, `Suffix`, `BaseID`, `AuthUsername`, `AuthPassword`, `EndpointStatus`, `Region`, and other operational fields.

Official docs show deployment stop/start/delete endpoints using plural `/deployments/...`, while the live OpenClaw runtime uses singular `/deployment/...`; keep both in mind if a route returns 404/405 during future testing.

Live balance validation on 2026-06-09 succeeded for Zeus Labs org scope using `THETA_ORG_ID`: response shape was `body.balances[]` with `org_id` and numeric `balance`.

## AI services coverage to preserve in future versions

A full Hermes tool/plugin version should eventually cover:

- Deployments: list/create/stop/delete
- Dedicated model templates: standard/custom
- Dedicated deployments listing
- Jupyter notebook listing
- GPU node / GPU cluster listing
- Persistent storage listing
- Agentic AI / RAG chatbot lifecycle
- On-demand model APIs: list/infer/status/poll/chat
- Dedicated inference endpoint: models/chat
- Theta Video APIs: list/upload/video/stream/ingestor

## Publishing for other Hermes users

Preferred public distribution options:

1. Publish to GitHub as a Hermes skill directory:
   ```bash
   hermes skills publish --to github --repo OWNER/REPO /path/to/theta-edgecloud-hermes-skill
   ```
2. Publish/submit to ClawHub if cross-agent distribution is desired. Current Hermes CLI support for ClawHub publishing prints a manual-submit notice, so use `https://clawhub.ai/submit` and clearly label the package as a Hermes port if ClawHub accepts it.
3. Users can install the full release archive to get `SKILL.md` plus helper scripts:
   ```bash
   curl -L https://github.com/zeuslabsllc/theta-edgecloud-hermes-skill/archive/refs/tags/v0.1.0.tar.gz -o theta-edgecloud-hermes-skill-v0.1.0.tar.gz
   mkdir -p ~/.hermes/skills/theta-edgecloud
   tar -xzf theta-edgecloud-hermes-skill-v0.1.0.tar.gz --strip-components=1 -C ~/.hermes/skills/theta-edgecloud
   ```
   Raw `SKILL.md` URL install/inspect is useful for previewing skill metadata but does not install bundled support scripts.

## When to build a Hermes plugin instead

A skill teaches Hermes how to use tools and scripts. A plugin/toolset is better if you want first-class tool calls like `theta_ondemand_chat`, `theta_deployments_list`, or `theta_video_create` to appear directly in Hermes tool schemas. Recommended path:

- v0.1: public Hermes skill with helper script and docs
- v0.2: add more helper script commands for deployments/video/agents
- v0.3: package as Hermes plugin/toolset or MCP server for native tool calls

---
> Source: [zeuslabsllc/theta-edgecloud-hermes-skill](https://github.com/zeuslabsllc/theta-edgecloud-hermes-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
