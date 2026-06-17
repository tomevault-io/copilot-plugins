## grok-account-bridge

> Use Grok through a locally signed-in X account using cookie credentials (AUTH_TOKEN/CT0) with strict preflight validation, secret-safe handling, and browser-first execution.


# 🪶 Grok Account Bridge — Local Signed-In X Session

Use this skill when the user explicitly wants **Grok via their local signed-in X account** and cookie-backed auth context.

## When to trigger

Trigger on prompts like:
- "use grok with my local account"
- "use auth_token and ct0"
- "use cookies from .claude/.env"
- "run through signed-in X account"

Do **not** auto-trigger for generic “use Grok API” requests unless the user asks for cookie/session mode.

## Default behavior

1. Validate local prerequisites (`bird` available)
2. Resolve credentials safely (`AUTH_TOKEN`, `CT0`)
3. Verify account context (`bird ... whoami`)
4. Execute via **browser-first Grok UI** path
5. Keep outputs concise and never expose credentials

## Auth resolution order (strict)

1. Existing process env:
   - `AUTH_TOKEN`
   - `CT0`
2. `~/.claude/.env` fallback (if env missing)
3. `~/.config/bird/config.json5` fallback (`auth_token` + `ct0`)

If either token is still missing, stop and return a precise remediation message.

## Mandatory preflight (must pass)

```bash
command -v bird && bird --version
```

```bash
bird --auth-token "$AUTH_TOKEN" --ct0 "$CT0" whoami
```

If preflight fails, do not attempt Grok execution. Explain exactly which layer failed:
- binary missing
- credentials missing
- credentials invalid/expired
- account/session mismatch

## Execution path

### Primary: browser-backed session (recommended)

Use in-app browser automation on the signed-in Grok UI:
1. Open browser
2. Navigate to `https://x.com/i/grok` (or current Grok URL)
3. Verify login state
4. Submit user prompt **without rewriting** unless asked
5. Return answer summary + any generated artifacts

This path is default because X internal endpoints can change.

### Optional: API mode (explicit opt-in only)

If and only if user asks for API mode:
- Use `XAI_API_KEY`
- Call `api.x.ai`
- Clearly mark output as **API mode**, not cookie-backed account mode

## Hard constraints

- Never print or log `AUTH_TOKEN`, `CT0`, or API keys
- Never persist secrets to files
- Never silently switch from cookie mode to API mode
- Never rewrite user prompt without user permission
- For state-changing actions, show draft and request confirmation first

## Fast diagnostics snippets

```bash
# presence checks only (safe)
[ -n "$AUTH_TOKEN" ] && echo "AUTH_TOKEN: set" || echo "AUTH_TOKEN: missing"
[ -n "$CT0" ] && echo "CT0: set" || echo "CT0: missing"

# cookie-session validation
bird --auth-token "$AUTH_TOKEN" --ct0 "$CT0" whoami
```

## Failure messaging template

Use this style:
- **Failed at:** credential validation
- **Reason:** `bird whoami` returned auth failure
- **Fix:** refresh X cookies and update `AUTH_TOKEN`/`CT0` in `~/.claude/.env`, then retry

## Security posture

- Principle of least exposure: only test what is required
- Output redaction by default
- Deterministic preflight before runtime actions
- No destructive action without explicit confirmation

---
> Source: [Sheshiyer/grok-account-bridge](https://github.com/Sheshiyer/grok-account-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
