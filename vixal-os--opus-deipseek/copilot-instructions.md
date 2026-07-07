## opus-deipseek

> Single-file (`bot.py`) multi-model Discord bot. EVA/MAGI-themed heuristic router over

# CLAUDE.md — Hydra (Opus-Deipseek)

Single-file (`bot.py`) multi-model Discord bot. EVA/MAGI-themed heuristic router over
**Claude (Balthasar) · DeepSeek (Melchior) · Gemini (Caspar)** plus open-weight heads
**Qwen (Rei) · GLM (Asuka)** on Fireworks and **Mistral (Mari)** on its own EU API. Two-tier memory,
prompt caching, bookclub mode, Mandarin (`!speak`) + French (`!french`) TTS,
research panel (`!research`), simulator mode (`!dummy` — base-model transcript completion),
and cost + carbon tracking.

This file is the working status + remaining-work tracker for the "Hydra Restructure"
spec. The full spec lives outside the repo (Sarah's Claude.ai artifact); this file is
self-contained — you don't need it to pick up the remaining work below.

## Run / verify
- `python bot.py` — needs `.env` (`DISCORD_TOKEN` + ≥1 model key) and `config.json`
  (`allowed_channels`, `default_model`).
- Syntax gate (no deps): `python -c "import ast; ast.parse(open('bot.py',encoding='utf-8').read())"`
- Import/logic test: `python -c "import bot"` works (deps installed). On Windows console
  prefix `PYTHONIOENCODING=utf-8` when printing bot output (emoji/CJK/IPA).

## Editing rules (load-bearing — read before touching bot.py)
- **Graceful degradation:** a missing API key disables ONLY that provider. The gating key follows
  the *active backend* (`provider.api_key_env`): by default `FIREWORKS_API_KEY` gates Qwen+GLM
  together and **Mistral has its own `MISTRAL_API_KEY`** (`api.mistral.ai`) — its flagship (Large 3)
  is on-demand-only on Fireworks, not serverless. `self_hosted`/`vertex` backends have no key and aren't
  key-gated (operator owns the local server / GCP ADC). Absent keys disable exactly those providers.
- **Config-driven registry (Phase 0).** `ProviderRegistry.from_config()` (built once in `__init__`
  from `config.json`'s optional `providers` block) wires every provider's client + backend. The 6
  `ModelProvider(...)` constants are the code-side default spec (pricing/energy/quirks +
  `sdk_type`/`api_key_env`/`base_url`/`backend`/`backends`); config supplies operator overrides
  (`enabled`/`backend`/`model`/`routing_tags`) + the `platform` toggle. **No `providers` block ⇒
  byte-for-byte the old behavior.** `routing_tags` is present but INERT (routing stays in
  `_estimate_confidence`).
- **`provider.name` is the canonical routing key.** `_select_model`, `_estimate_confidence`,
  `panel_members`, cost, and persistence all branch on it. NEVER rename it. (`provider.id` — the
  lowercase config key — is separate; it keys `config.json` + `self.clients`.) Display names and
  command aliases are a separate theme layer on top (see naming below).
- **Dispatch** routes on `provider.sdk_type` (`anthropic` | `openai_compatible` | `gemini`), set by
  the registry. Claude → anthropic SDK; Gemini bookclub → native `cachedContents` (or Vertex
  `CachedContent` when `backend="vertex"`); everything else → the OpenAI-compatible shim
  (`_generate_openai_compatible_response`) via `self.clients[provider.id]`. Per-provider wiring lives
  in the registry, not `__init__`.
- **Model slugs drift** — verify in the live libraries. Fireworks (verified 2026-06):
  `qwen3p7-plus`, `glm-5p2`, `deepseek-v4-pro` (under `accounts/fireworks/models/`). Mistral on
  its own API uses `mistral-large-latest` (→ Mistral Large 3, released 2025-12, 675B/41B MoE,
  Apache-2.0; priced $0.50/$1.50 per Mtok as of 2026-06 — a ~4× drop from Large 2). ⚠️ Large 3 is
  on Fireworks only as **on-demand/dedicated** (`mistral-large-3-fp8`), NOT serverless — so the
  own-API route stays correct AND is now the cheap one, not just the green one.
- New-provider **pricing + energy constants are estimates** flagged `VERIFY` in comments —
  confirm against the Fireworks pricing page before trusting `!cost` $ figures.

## Naming theme — now a toggleable **theme layer** (`config.json` top-level `"theme"`)
Themes are a **display-only skin**: `Flavor`/`Theme`/`THEMES` (just after `SIM_PROVIDER`) map each
`provider.id` → themed display name + command aliases + a persona note. Read in
`ProviderRegistry.from_config` (mirrors the `platform` toggle), applied onto `provider.display_name`
and the `self.alias_to_flag` map built in `__init__`. **The canonical `provider.name`, the
`**[Claude]**` label, and `MODEL_LABEL_NAMES` never change** — a Judah/Gold-Head turn is still
labeled `[Claude]` on the wire. Canonical bare prefixes (`!claude`/`!deepseek`/…) + `!think` work in
every theme; the theme only *adds* its flavor aliases. Three sets ship (default **`eva`** ⇒
byte-for-byte the old behavior):
- **`eva` — EVA/MAGI** (default). Trinity Balthasar/Melchior/Caspar; pilots Mistral=Mari (`!mari`),
  Qwen=Rei (`!rei`), GLM=Asuka (`!asuka`); sim=Dummy Plug (`!dummy`).
- **`isaic` — ISAIC** ("International System of AI Coopertition"), the twelve tribes: Judah=Claude
  (`!judah`), Joseph=Gemini (`!joseph`), Zebulun=DeepSeek (`!zebulun`), Naphtali=Mistral
  (`!naphtali`), Benjamin=Qwen (`!benjamin`), Gad=GLM (`!gad`), Levi=sim (`!levi`). **Now in code**
  (was docs-only); it's also the Slack bot's default skin (Phase 4).
- **`nightvale` — the five heads of Hiram McDaniels** (+ residents): Gold=Claude (`!gold`),
  Blue=Gemini (`!blue`), Green=DeepSeek (`!green`), Violet=Mistral (`!violet`/`!purple`), Gray=Qwen
  (`!gray`/`!grey`); plus Carlos=GLM (`!carlos`) and the Faceless Old Woman=sim (`!faceless`).

---

## Status — 2026-06-27

### ✅ Built (validated offline, needs live-key smoke tests)
- **Phase 7 — simulator mode, rung 1 (2026-06-27).** The **Dummy Plug** (`!dummy`/`!sim`): a base
  model continues the channel as an IRC/script transcript via `/completions` instead of chatting.
  `ModelProvider.completions_mode` + `sim_sampler` (§9.1/9.3); `_format_transcript` /
  `_parse_transcript_turn` / `_generate_simulator_response` (one dispatch branch ahead of the
  `sdk_type` cases); cost **and** carbon ride the same `record_usage` path as the chat heads (§9.2).
  Config-driven (`providers.sim.{enabled,base_url,model,api_key_env,sampler,search}`); **off by
  default** and **override-only** (excluded from the argmax; reachable via `!dummy` or `!prefer sim`
  — the spec's "designated channel"). Behavior-preserving on the default config (46-check offline
  harness + adversarial multi-agent review, findings folded in). **Scope (Sarah's call):** rung 1
  only — rungs 2–3 (ambient turn-taking, webhook personas; "the loom") were deliberately NOT built,
  and simulator mode is intentionally kept out of Phase 4 / lab planning. ⚠️ Owes one live smoke
  test against a real base endpoint.
  - **Query-driven search grounding (§9.2/§9.5, wired 2026-06-27).** A base model can't tool-call,
    so `_generate_simulator_response` now detects a search intent in the latest user turn
    (`_sim_search_query`: explicit `[search: …]` directive or a leading-question/lookup-cue
    heuristic) and **prepends** `_search_for(provider, query).text` to the transcript preamble —
    the sim analogue of the chat heads' tool-result append. **Off by default** (`sim_search`,
    flip via `providers.sim.search=true`) and gated on a real backend being available
    (`_search_backend_available`), so the default Dummy Plug never auto-searches. `SIM_PROVIDER`'s
    `search_backend` is back to `"tavily"`. Pasted-URL grounding still folds in regardless via
    `_augment_with_url_extracts`. Offline-validated (31-check `test_sim_search.py`).
- **Phases 0 + 2 + 3 — registry + Gemini Vertex + DeepSeek/Mistral backend toggles (2026-06-24).**
  Config-driven `ProviderRegistry` (`sdk_type` dispatch; `config.json` `providers`+`platform`),
  behavior-preserving on the default config (24-check offline harness passes). Gemini `vertex`
  backend + DeepSeek `fireworks`/`self_hosted` + Mistral `together`/`self_hosted` are code-complete
  with a `local` cost mode; the non-default backends ⚠️ owe live smoke tests. See the table +
  per-phase sections above.
- **Bookclub Gemini cache cost controls (2026-06-24).** Context-cache storage bills per token-hour
  for the whole TTL (~$11.6 for a 452k-token fic at the old fixed 24h), so: TTL is now a **6h
  sliding window** (`GEMINI_CACHE_TTL_HOURS` env knob; `_refresh_gemini_cache` PATCHes the expiry
  forward once past halfway → active discussion never expires mid-conversation, idle dies ~6h after
  last use), and **`!uncache`** drops just the Gemini cache while keeping the book loaded for all
  models (vs `!unload`, which removes the shared `ReadingMaterial` for everyone). Idle caches also
  self-expire, so day-to-day neither is needed.
- **Phase 1 — Qwen/GLM on Fireworks, Mistral on `api.mistral.ai`.** Qwen+GLM share one
  Fireworks key (cached input = 0.5×input); Mistral is on its own EU API (`MISTRAL_API_KEY`)
  because Mistral Large 3 isn't on Fireworks serverless — a happy accident: EU-resident + France
  ~nuclear grid (grid≈20 in `!cost`). Graceful degradation, EVA aliases, per-provider personas,
  in `!cost` and the `!research all` panel.
- **Routing — intentional deviation from the spec.** The spec said new heads "override-only."
  Per Sarah's call that cheap models should earn routing: **Qwen is in the auto-router as the
  cheap coder/mathematician** (wins routine code/math, hands complex/careful work back to
  Claude), **Mistral gets a narrow French-intent nudge**, **GLM stays override-only**.
  ⚠️ Do NOT "fix" this back to all-override-only — it's deliberate.
- **Beyond the spec (Sarah's additions):**
  - `!research all` — opt-in 6-model panel (plain `!research` stays the lean core trio).
  - **Energy/CO₂ in `!cost`** — per-provider Wh (`est_wh_per_1k_tokens`) × per-provider grid
    intensity (`grid_gco2_per_kwh`, follows the *endpoint* not the brand), plus a separate
    amortized-training line (`train_tco2e` over `MODEL_LIFETIME_TOKENS`). Env knobs:
    `GRID_GCO2_PER_KWH`, `MODEL_LIFETIME_TOKENS`, `AMORTIZE_TRAINING`. All flagged
    order-of-magnitude; never used for routing.
  - **`!french` tutor** — inverse of the Mandarin path: Azure `fr-FR-DeniseNeural`, Mistral-native
    G2P returning text+IPA+liaison note, IPA is display-only (no forced phonemes — Azure infers),
    inline `[[french:..]]` for models. Shares Azure config with `!speak`.
  - **Per-server system prompts (config.json `servers`, keyed by guild id) + runtime editors.** The
    system-prompt template has two placeholders filled per guild in `_build_system_prompt(...,
    guild_id=)`: `{server_context}` (the "context about this server" block) and `{plurality_section}`
    (the whole PluralKit block, dropped when a server sets `pluralkit: false`). Lookup order:
    `servers[<guild_id>]` → `server_default` → the `BotConfig` defaults. **The committed
    `default_server_context` is now GENERIC** — it tells the model it knows nothing about who's
    there and must not invent facts (secrets hygiene: no real names/servers in the repo; personal
    context lives only in git-ignored config.json or is set at runtime). `default_plurality_section`
    default is still `pluralkit=true`. The tidy-spacing `re.sub` runs whenever a guild deviates from
    the default. Config entries are validated/normalized by `_coerce_server_cfg` (non-dict → `{}` +
    warn; a quoted `"pluralkit"` → real bool + warn) so a config typo warns at startup instead of
    crashing `_build_system_prompt` on every message and silently muting the guild. `{server_context}`
    is substituted LAST so a custom context can't re-expand a placeholder token. `on_ready` prints
    each joined guild's name+id. **Runtime editors** (`!channels`, `!server_context`; admin/owner-only
    via `_can_edit_config`) mutate the in-memory state AND persist through `_save_config()`, which
    re-dumps `self._raw_config` to config.json (git-ignored) preserving all other keys — so a Discord
    edit and a hand-edit stay in sync. Rationale: the *old* prompt hardcoded "Sarah's projects /
    neuroscience / distributed databases" + an always-on PluralKit section, which made the bot invent
    plural-alter histories on servers that don't use PluralKit.
  - **Working-note extraction hardened.** All 5 generators + both web-search paths now call one
    `_extract_notes(text, guild_id)` helper (compiled `_NOTE_RE`, **case-insensitive** so
    `[Note:]`/`[NOTE:]` are caught, idempotent so it doubles as a safety net). Fixes a real leak:
    `_web_search` (Claude native) and the grounded-Gemini `!search` branch returned model text
    straight to Discord WITHOUT stripping `[note:]`, so a note tacked onto a `!search` answer leaked
    into the message.

### Remaining from the restructure spec
| Phase | Status | Priority |
|---|---|---|
| 0 — Provider registry refactor (`sdk_type`, config-driven) | ✅ **done** — `ProviderRegistry`, config `providers`+`platform`, `sdk_type` dispatch (behavior-preserving; offline-validated) | — |
| 1 — Mistral/Qwen/GLM on Fireworks | ✅ done (with routing deviation) | — |
| 2 — Gemini Developer-API default | ✅ already the default (operator: keep billing on) | — |
| 2 — Gemini **Vertex** backend | ✅ **code-complete** — `_generate_gemini_vertex_response` + `_ensure_gemini_vertex_cache` + backend-tagged caches; ⚠️ UNVERIFIED (needs GCP ADC) | Optional / lab-only |
| 3 — Provider backend toggles: DeepSeek (`api`/`fireworks`/`self_hosted`) **+ Mistral** (`api`/`together`/`self_hosted`) | ✅ **code-complete** — toggles in registry + `local` cost mode; `api` modes live, `fireworks`/`self_hosted`/`together` ⚠️ UNVERIFIED | Med — lab route |
| 4 — Slack platform abstraction | ❌ not built | **High — the big one** |
| 5 — Hermes delegation bridge | ❌ not built | Optional / lab-only |
| 6 — Ephemeral big-model co-op (405B base) | ❌ not built | Optional / lab-only |
| 7 — Simulator mode (§9, rung 1: transcript completion) | ✅ **code-complete** — Dummy Plug (`!dummy`/`!sim`), config-driven, override-only, cost+carbon tracked (offline-validated, 46-check harness); ⚠️ owes a live smoke test against a real `/completions` endpoint. Rungs 2–3 (ambient/personas) intentionally NOT built. | Done (fun) |

#### Phase 0 — Provider registry refactor — *✅ done (this session)*
`ProviderRegistry.from_config()` (right after the provider constants in `bot.py`) builds every
provider's client + backend from `config.json`. The 6 `ModelProvider(...)` constants stay as the
code-side default spec (pricing/energy/quirks + `sdk_type`/`api_key_env`/`base_url`/`backend`/
`backends`); config's optional `providers` block overrides `enabled`/`backend`/`model`/
`routing_tags`, and a top-level `platform` field is read+stored (Slack is Phase 4). **Hybrid by
design** — the richly-commented constants did NOT move into JSON; config is operator overrides only.
**No `providers` block ⇒ byte-for-byte the old behavior** (offline-validated: enabled set, dispatch,
cost, routing all unchanged). `__init__` is slimmed to bind registry results (`self.providers`,
`self.<id>_provider`, `self.clients`, `self.openai_compatible_clients`); dispatch switches on
`provider.sdk_type`; `pref_map`/`!prefer` are registry-built (so `!prefer qwen|glm|mistral` now work
— the one intentional additive change). `routing_tags` exists but is INERT. Package split deferred
to Phase 4.

#### Phase 2 (Vertex) — *✅ code-complete, ⚠️ unverified (this session)*
Developer API stays the default + no-train **as long as billing stays enabled on the Google project**
(operator setting — Sarah is paying). `gemini.backend = "vertex"` now routes to
`_generate_gemini_vertex_response()` + `_ensure_gemini_vertex_cache()` (lazy `vertexai` import;
`google-cloud-aiplatform` is an OPTIONAL, commented dep). **Cache handles are backend-tagged**
(`Gemini:developer_api` vs `Gemini:vertex` via `_gemini_cache_key()`) so a `cachedContents` id is
never reused against Vertex `CachedContent`; `_drop`/`_reconcile` handle both + the legacy untagged
key. ⚠️ **OWES A LIVE SMOKE TEST** — needs GCP ADC (`GOOGLE_APPLICATION_CREDENTIALS`) +
`GOOGLE_CLOUD_PROJECT`; the exact vertexai SDK surface (grounding ctor, `from_cached_content`) may
need a tweak per SDK version. `_reconcile_gemini_caches` still sweeps only developer-API caches
(Vertex orphans rely on the per-teardown drop — `TODO(vertex-reconcile)`).

#### Phase 3 — Provider backend toggles (DeepSeek + Mistral) — *✅ code-complete, ⚠️ unverified*
Both toggles are built in the registry (`backends` tables on the constants; selected via
`providers.<id>.backend`). `api` modes are live + unchanged; `fireworks`/`together`/`self_hosted`
are wired but ⚠️ **OWE LIVE SMOKE TESTS** (Fireworks/Together keys, a local vLLM). `self_hosted`
sets `cost_mode="local"` + `supports_server_cache=False` → `get_cost()` returns $0 and `!cost`
labels the line "local (electricity only)" while still showing the 🌱 energy/CO₂ (tokens counted on
the self-host grid). DeepSeek still defaults to its China `api` for the Discord bot (locked
deviation). The per-backend reference below still applies (values flagged `VERIFY` are estimates):

**DeepSeek** stays on its China `api` for the Discord bot (Sarah's call — cheap, and "the Chinese can
have the shitposts"). To add the toggle: `providers.deepseek.backend = api | fireworks | self_hosted`.
- `fireworks` → `accounts/fireworks/models/deepseek-v4-pro`, shares `FIREWORKS_API_KEY`, keep
  per-token cost at Fireworks rates (~1.74/3.48), server cache at 50%. Update its
  `grid_gco2_per_kwh` to ~400 (US) when on this backend.
- `self_hosted` → local vLLM/Ollama base_url; set `supports_server_cache=False`; add a `local`
  cost mode in `_record_*_usage()` (skip per-token $, label the turn `local`); V4-Flash or 4-bit
  for single-GPU.

**Mistral** is hardcoded to `api.mistral.ai` (set up this way because Mistral Large 3 isn't on
Fireworks *serverless* — `mistral-large-3-fp8` is on-demand/dedicated only). Same
toggle shape: `providers.mistral.backend = api | together | self_hosted`.
- `api` (current) → `api.mistral.ai`, `MISTRAL_API_KEY`, model `mistral-large-latest` (→ Large 3,
  $0.50/$1.50 per Mtok as of 2026-06). EU-resident + France ~nuclear grid (`grid_gco2_per_kwh=20`).
  **Best default for the Discord bot — keep it** (now the cheap route too, ~4× under Large 2).
- `together` → Together AI serverless (US) *does* host Mistral Large 3 — use for US residency / one
  fewer bill; set `grid_gco2_per_kwh≈400` + Together's rates. (Fireworks only has it on-demand —
  i.e. rent a dedicated GPU — not worth it for a chat bot.)
- `self_hosted` → Apache-2.0 weights, but Large 3 is 675B (multi-GPU). For single-GPU, self-host a
  smaller Mistral (Ministral 3B/8B/14B, Magistral Small 24B); reuse DeepSeek's `local` cost mode.

#### Phase 4 — Slack platform abstraction — *TODO, the big remaining chunk*
Introduce a `ChatPlatform` protocol; make `ClaudeBot`/`ConversationManager` depend on it instead
of `discord.py` directly. Extract `DiscordAdapter` (wrap existing logic, don't rewrite behavior);
build `SlackAdapter` on `slack_bolt` in **Socket Mode** (no public URL). Specifics:
- Threads → `thread_ts`; 👍/👎 calibration → `reaction_added`/`reaction_removed`; files →
  `files_upload_v2`; `!claude`-style commands → slash commands or `app_mention` parse.
- **Re-key persistence** `memories.json` on `(platform, team/guild, channel)` + a migration that
  namespaces existing Discord keys under `discord:<guild>` (else Discord/Slack clobber each other).
- **Formatting:** Slack is mrkdwn + Block Kit, not Markdown — audit every emit (code fences, bold,
  links, `!help` tables). LaTeX→PNG and the TTS MP3 paths are fine (image/file upload).
- **De-dupe** Slack's event retries on `event_id`/`client_msg_id`.
- Ship a Slack app manifest (scopes + event subs + slash commands). Tokens: `SLACK_BOT_TOKEN`
  (`xoxb-`), `SLACK_APP_TOKEN` (`xapp-`).
- This is where the **ISAIC naming skin** activates (Slack = lab bot).

#### Phase 5 — Hermes delegation bridge — *optional, lab-only*
Co-located Hermes Agent (separate service, thin bridge — do NOT merge codebases). Route `!research`
or a new `/task` to a local Hermes instance, stream the trajectory back via existing `send_text`/
`send_file`. **Egress discipline is a security rule, not a convenience:** force Hermes onto the same
hardened endpoints (Fireworks US/ZDR, self-hosted DeepSeek, Vertex); gate genuinely patentable
prompts to self-hosted only. Optional cron delivery into the lab channel. Hydra must still run
fully with Hermes absent.

#### Phase 7 — Simulator mode (§9, rung 1) — *✅ code-complete, ⚠️ unverified (2026-06-27)*
A second **generation mode** behind the same dispatch (NOT a second bot): `completions_mode=True`
routes a provider to `_generate_simulator_response()` (transcript completion) instead of the chat
path. The whole feature is three methods (§9.3): `_format_transcript` renders channel history as a
`<speaker> body` IRC log (user turns carry `Name:`, bot turns `[Model]`; a leading `[replying to …]`
block is dropped at the block level so embedded `]`/newlines can't corrupt the speaker line) + a
dangling `<Dummy>` continuation cue; `client.completions.create` continues it; `_parse_transcript_turn`
cuts back to one line (server `stop` + a client-side `<name>`-only cut — deliberately NOT `[name]`,
which would eat code/citation lines). `SIM_PROVIDER` ("Dummy"/"Dummy Plug", `id="sim"`) ships
`enabled=False` + `api_key_env=""` (self-hosted, keyless), so the registry leaves it OFF unless
config opts in — the one place `config_enabled` now defaults to the constant's `enabled` instead of
`True` (a no-op for the 6 always-on heads). **Override-only** is enforced in `_select_model`
(`auto_pool` drops `completions_mode`), with one deliberate exception documented in code: if the
base model is the *only* enabled provider (a sim-only box), plain messages route to it via the
`len(enabled)==1` shortcut. Sampler knobs (§9.3) split standard params (on the call) vs
`top_k`/`min_p`/`top_a`/`repetition_penalty` (in `extra_body`); unknown `providers.sim.sampler` keys
warn at startup. ⚠️ **OWES A LIVE SMOKE TEST** — stand up any `/completions` base model (a small
GGUF/Ministral base on vLLM or llama.cpp suffices — no 405B), set
`providers.sim = {enabled:true, base_url, model}`, then `!dummy <prompt>`; confirm the `[Dummy]`
turn lands in `!cost` with tokens + 🌱. **Query-driven search grounding** (§9.2/§9.5) is now wired
behind `providers.sim.search=true` (off by default): `_sim_search_query` detects a `[search: …]`
directive or a leading-question/lookup cue and `_generate_simulator_response` prepends the
`_search_for` snippets to the preamble (gated on `_search_backend_available` so no sentinel leaks).
Rungs 2–3 (ambient turn-taking, webhook personas) and any Phase 4 / lab-adjacent simulator work are
intentionally out of scope.

### Cross-cutting follow-ups
- Verify **pricing**: Fireworks for `qwen3p7-plus` / `glm-5p2` (confirmed live 2026-06), and
  `console.mistral.ai` for `mistral-large-latest` (updated 2026-06 to Large 3 = $0.50/$1.50; reconfirm
  against the console). The **energy** constants (`est_wh_per_1k_tokens`, `train_tco2e`) are
  order-of-magnitude (Mistral `train_tco2e` is still the Large-2 LCA — Large-3's isn't published).
- Live smoke tests still owed: `!mari`/`!rei`/`!asuka` round-trips, `!french bonjour` (Azure
  fr-FR synth) and `!french how do you say …` (Mistral G2P), inline `[[french:..]]`. **Plus the
  new backends (Phase 2/3):** Gemini `vertex` (GCP ADC + `google-cloud-aiplatform`); DeepSeek
  `fireworks` (`!deepseek`) + `self_hosted` (local vLLM → shows `local` in `!cost`); Mistral
  `together` (Together key) + `self_hosted`. Verify the Together Mistral-Large-3 slug + Fireworks
  DeepSeek/Together pricing (flagged `VERIFY`).
- **Phase 7 simulator:** `!dummy <prompt>` against a real `/completions` base endpoint (transcript
  continuation, tokens + carbon in `!cost`); `SIM_PROVIDER` pricing/energy are `VERIFY` placeholders.
  Also smoke-test search grounding: `providers.sim.search=true` + a Tavily key, then a `!dummy`
  turn whose latest line is a question / `[search: …]` — confirm the snippets land in the preamble.
- Operator: keep Fireworks prepaid balance topped up (auto-reload) and Gemini billing current.

---
> Source: [VIXAL-OS/Opus-Deipseek](https://github.com/VIXAL-OS/Opus-Deipseek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
