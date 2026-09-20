## agento

> Follow the shared rules in [AGENTS.md](../AGENTS.md) and the development workflow

# agento repository instructions

Follow the shared rules in [AGENTS.md](../AGENTS.md) and the development workflow
in [CONTRIBUTING.md](../CONTRIBUTING.md). agento is a Python library embedded in a
host application, with a Pydantic-only core and optional provider, MCP, SQL, and
tracing adapters. Core/OpenAI/MCP support Python 3.10+; LiteLLM requires 3.11+.

For reviews and fixes, prioritize these observable contracts:

- A durable event must not become visible before its matching snapshot commits.
- Session-tip creation must reject stale writers and active-turn conflicts;
  terminal records must not be mutated or counted twice.
- Cancellation/recovery must close owned generators and transports. An external
  call with an unknown outcome requires reconciliation before side-effect retry.
- Serialized state must not contain credentials or depend on restoring a Python
  callable. The host owns auth, tenancy, sandboxing, retention, and idempotency.
- Optional imports must stay out of the minimal core import path.

Use focused patches and meaningful failure-path tests. Check memory and SQL
implementations together when changing persistence. Run Ruff, strict mypy, and
the affected tests; run the complete contribution checks before requesting merge.
Tests and examples use deterministic/offline adapters and need no provider key.
Keep README examples brief and detailed usage under `docs/`.

Review performance claims against `docs/token-efficiency.md` and its reproducible
measurement script. Distinguish request-context reduction from whole-task token
use, provider billing, latency, and answer quality. Report concrete file/line
evidence, reproduction, and impact; avoid unsupported production-readiness claims.

Do not weaken checks, broaden workflow permissions, or add package publication
credentials as part of an unrelated fix. Do not commit IDE/assistant state,
generated audit output, local artifacts, or secrets. Agent-authored pull requests
follow the same protected-main checks and maintainer review as other changes.

---
> Source: [GtechGovind/agento](https://github.com/GtechGovind/agento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
