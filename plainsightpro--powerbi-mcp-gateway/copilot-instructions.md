## powerbi-mcp-gateway

> A remote MCP server that gives any MCP client (Claude Code, Claude Desktop, claude.ai, VS Code,

# powerbi-mcp-gateway - AI coding assistant guide

A remote MCP server that gives any MCP client (Claude Code, Claude Desktop, claude.ai, VS Code,
Cursor) per-user access to Power BI semantic models, with the organisation's know-how ("skills")
kept on the server and DAX generation on a Foundry (Azure OpenAI) deployment. See README.md for the
architecture; this file is for working in the code.

## Layout

| Path | Role |
|---|---|
| `powerbi_mcp/server.py` | The MCP surface: AzureProvider OAuth proxy, tool signatures, prompts, resources, `/healthz`, lifespan |
| `powerbi_mcp/gateway.py` | What the tools do: caches, pooled HTTP clients, error mapping, generate -> execute -> repair; testable without OAuth |
| `powerbi_mcp/observability.py` | User key from the access token; one log line per tool call, never query text or rows |
| `powerbi_mcp/state_store.py` | OAuth proxy state in an Azure Table (encrypted), when `PBIMCP_STATE_STORAGE_ACCOUNT` is set |
| `powerbi_mcp/validate.py` | `python -m powerbi_mcp --check-skills [DIR]`: skills folder checks used by CI and the deploy script |
| `powerbi_mcp/config.py` | Settings from `PBIMCP_*` env vars / `.env` |
| `powerbi_mcp/fabric.py` | Fabric REST: workspaces and semantic models the user can open |
| `powerbi_mcp/hosted_mcp.py` | Client for Microsoft's hosted Power BI MCP (schema, DAX execution, report metadata) |
| `powerbi_mcp/catalog.py` | Curated catalog (`skills/catalog.yaml`) merged with the user's accessible models |
| `powerbi_mcp/dax_generator.py` | gpt-5 DAX generation + one repair round, strict JSON output |
| `powerbi_mcp/skills.py` | Loads the skills folder once at startup |
| `powerbi_mcp/memory.py` | Memories: notes per semantic model in a key-value store (JSON files locally, an Azure Table with the OAuth state) |
| `skills/` | Example skills (fictional Contoso Retail). Real skills live in a private folder: `PBIMCP_SKILLS_DIR` / `-SkillsDir` |
| `deploy/deploy_to_azure.ps1` | Idempotent Azure deploy (Entra app, Foundry, ACR, Container Apps); `-Profile` JSON, `-EngineImage` |
| `deploy/build_context.ps1` | Stages only runtime code + selected skills, or `FROM <engine image>` + skills |
| `deploy/profiles/example.json` | The keys a deployment sets (a test keeps them aligned with the script's parameters), fictional values |
| `.github/workflows/release.yml` | Tag `vX.Y.Z` publishes `ghcr.io/plainsightpro/powerbi-mcp-gateway:X.Y.Z` |
| `scripts/smoke_test.py` | Headless end-to-end check of everything behind the OAuth proxy |
| `scripts/check_gateway.py` | Real-OAuth check of a deployed gateway |
| `docs/` | Consumer walkthroughs per client, usage guidelines, troubleshooting, administration, skills authoring |
| `tests/` | pytest, no network (httpx MockTransport, stub Responses client, in-memory key-value store) |

## Commands (Windows, PowerShell or Git Bash)

```
uv sync                                                 # .venv from uv.lock, including the dev group
uv run ruff check . && uv run ruff format --check . && uv run pyright
uv run pytest -q
uv run python -m powerbi_mcp                            # http://localhost:8000/mcp
uv run python scripts/smoke_test.py "question"
.\deploy\deploy_to_azure.ps1 -AcrName <yourUniqueRegistry>                 # example skills, source build
.\deploy\deploy_to_azure.ps1 -Profile C:\deployments\<org>\deploy\profile.json   # private skills + names
```

`.env` (git-ignored) carries the Entra client id/secret and the Foundry endpoint; copy from
`.env.example`. The smoke test needs `-PreauthorizeAzureCli` done once by the deploy script.

## Design rules

- Every Power BI call carries the signed-in user's token (`EntraOBOToken`); never add a service
  principal path. Premium Per User terms forbid shared identities for multi-user apps and RLS
  would not apply.
- The OAuth proxy issues its own tokens; MCP clients register dynamically (DCR, CIMD). Keep
  `forward_resource=False`: Entra's v2 endpoint rejects an RFC 8707 `resource` next to `scope`.
- Do not put downstream (Fabric) scopes in `additional_authorize_scopes`; admin consent on the
  app's delegated Power BI permissions is what makes the OBO exchange work.
- Text answers on structured hosted-MCP tools are errors (that is how the hosted server reports a
  DAX syntax error); `generate_dax` then gets one repair round with the engine message.
- The engine carries no domain vocabulary. Tool names are generic (`get_business_context`,
  `get_recipe`), MCP prompts are generated one-per-recipe from the skills folder, and tests assert
  structure only (recipes on disk == prompts; a fixture with an invented recipe must work). Never
  hard-code a recipe name, a measure or a company term in `powerbi_mcp/` or `tests/`.
- Keep public examples fictional (Contoso Retail) and free of any real organisation's terms. See
  `docs/private-skills.md` for one codebase with private skill folders and deployment profiles.
  Skills are deployment-wide, not permission-filtered.
- Memories follow the model and never carry an access list: every read or write goes through
  `Gateway.require_access`, which is the schema fetch with the user's own token (a 401/403 there is
  the answer). Do not add a cheaper check (the Fabric listing includes models a Viewer cannot
  query) and do not cache a "may see" flag anywhere but the schema cache.
- One replica. The proxy's state is in an Azure Table when `PBIMCP_STATE_STORAGE_ACCOUNT` is set
  (the deploy script sets it); scaling out additionally needs `stateless_http=True`.
- Fixtures in `tests/` use invented names (a fictional retailer's tables); never a real model's.

## Deploy gotchas already handled in the script (do not "simplify" them away)

- A PowerShell helper named `Az` calling `& az` recurses into itself; helpers are `Invoke-Az*` and
  call the resolved `az.cmd`.
- Azure Policy may require an `Owner` tag on resource groups (`-OwnerTag`).
- Graph refuses `preAuthorizedApplications` in the PATCH that creates the scope it references.
- Admin consent right after `az ad sp create` races replication; the script retries.
- `az acr build` log streaming crashes on Unicode on Windows (`az.cmd` runs Python with `-I`, so
  `PYTHONUTF8` cannot help); the script queues with `--no-logs` and polls the run.
- JSON bodies for `az rest --body` go through a temp file (`@file`), never inline.
- `azure.identity.aio` needs `aiohttp`; it is a declared dependency for that reason.
- The registry has no admin user; the container app pulls with `--registry-identity system`. The
  storage account has shared keys disabled; the app reaches its table with its identity.
- `PBIMCP_BASE_URL` is computed from the environment's `defaultDomain` before the app is created,
  so the first revision already has the right redirect URI.

---
> Source: [PlainsightPro/powerbi-mcp-gateway](https://github.com/PlainsightPro/powerbi-mcp-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
