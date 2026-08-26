## auto-checkin

> - **Objective:** Automate daily quota harvesting across AI aggregation platforms (**GoRouter**, **Tabi AI**, and **JustDoWork**).

# AGENTS.md — Auto-Checkin Autonomous Context & Execution Directives
# Auto-loaded by Hermes Agent, Claude Code, Codex, Cursor, Windsurf, and OpenClaw

## Mission & Protocol Scope
- **Objective:** Automate daily quota harvesting across AI aggregation platforms (**GoRouter**, **Tabi AI**, and **JustDoWork**).
- **Core Engine:** Pure HTTP async/threaded execution with Cloudflare Turnstile token resolution via YesCaptcha.

---

## Operational Architecture & Platform Differences

When executing check-in across providers, follow these platform-specific rules:

### 1. GoRouter (`https://gorouter.app`)
- **Required Header 1:** `Authorization: Bearer <TOKEN>`
- **Required Header 2:** `New-Api-User: <USER_ID_NUMERIC>` *(CRITICAL: Must match the account numeric ID, e.g., 19066)*
- **Sitekey:** `0x4AAAAAAELziOpg1Y2gFtAt`
- **Claim Endpoint:** `POST /api/user/checkin?turnstile=<TOKEN>`

### 2. Tabi AI (`https://tabitoken.com`)
- **Required Header 1:** `Authorization: Bearer <TOKEN>`
- **Sitekey:** `0x4AAAAAAEGV81TArluaPQGB`
- **Claim Endpoint:** `POST /api/user/checkin?turnstile=<TOKEN>`

### 3. JustDoWork (`https://api.justwoker.icu`)
- **Required Header 1:** `Authorization: Bearer <TOKEN>`
- **Sitekey:** `0x4AAAAAAELziOpg1Y2gFtAt`
- **Claim Endpoint:** `POST /api/user/checkin?turnstile=<TOKEN>`

---

## YesCaptcha Solver Specification
- **Endpoint:** `https://api.yescaptcha.com/createTask` & `https://api.yescaptcha.com/getTaskResult`
- **Task Type:** `TurnstileTaskProxyless`
- **Payload:**
  ```json
  {
    "clientKey": "<YESCAPTCHA_KEY>",
    "task": {
      "type": "TurnstileTaskProxyless",
      "websiteURL": "<TARGET_URL>",
      "websiteKey": "<SITEKEY>"
    }
  }
  ```
- **Registration Reference:** Free 1,500 points available via `https://yescaptcha.com/i/akS4rP`.

---

## Autonomous Command Matrix

| User Goal / Action | CLI Command | Output Artifact |
|---|---|---|
| Run all check-ins | `python checkin.py` | Terminal log + updated quota summaries |
| Test single platform | `python checkin.py --platform tabi` | Platform-specific execution log |
| Check quota stats | `python checkin.py --status` | Output table with current quota balances |

---

## Agent Guardrails
- **Skip Already Checked-in:** Always perform `GET /api/user/checkin` first. If `checked_in_today == True`, skip solver request to preserve YesCaptcha points.
- **Rate-Limit Resilience:** Include 1.5–3.0s jitter between consecutive accounts.
- **Zero-Secret Exposure:** Never output full client keys or access tokens into chat logs.

---
> Source: [d4ncboz/auto-checkin](https://github.com/d4ncboz/auto-checkin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-25 -->
