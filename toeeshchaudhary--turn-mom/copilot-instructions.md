## turn-mom

> Build the SFT **dataset** for a 3-suggestion `RecommendationResponse` CSS agent for New American Funding

# CLAUDE.md — ChadGPT / turn_mom (dataset pipeline)

Build the SFT **dataset** for a 3-suggestion `RecommendationResponse` CSS agent for New American Funding
(NAF): given a CONTEXT block, the target output is exactly 3 pick-able SMS replies + confidence. This repo
is now **data only** — clean → rehydrate → build tasks → label with the teacher → audit → emit
`data/sft/{train,val}.jsonl`. Read `README.md` for the runbook. (Training/serving/eval code was removed;
this is the dataset half.)

## What the labels target
- **Shape:** `system`(css prompt) + `user`(CONTEXT block) → `assistant` `{"recommendations":[{suggested_message,confidence}x3]}`.
- **Not** a single free-text reply (that was the MVP). The product is always 3 suggestions with stage logic
  (qualifying / confirming / eligible / ineligible + edge cases greeting/casual/logistics/offtopic).
- The authoritative behavior spec is `prompts/css_system_prompt.txt` (voice rules + stage rules + guardrails);
  `prompts/css_maos_system_prompt.txt` is the emotion-first MAOS variant, `prompts/css_teacher_prompt.txt`
  the labeler instruction.

## Teacher (synthetic labeler)
- Local `nvidia/Llama-3.3-70B-Instruct-FP8` (pre-quantized) on the GH200, vLLM :8001, prefix-caching on
  (gated on HF — set `HF_TOKEN`). `serve/serve_teacher.sh` omits `--quantization` for pre-quantized
  checkpoints; set `QUANT=fp8` for a bf16 model.

## Data (R2 bucket `chadgpt-data`)
- **`Bonzo/` (21,307 json)** — real Agent↔Client **SMS**. `Bonzo/Clean Redacted/` has PII tokens
  (`{NAME_GIVEN}` …). **This is the voice source** (already texting register).
- **`{March,April,May} Transcripts/` (~4,800 txt)** — noisy ASR **call** transcripts (CSS↔customer).
  Used only for flow/stage coverage; teacher rewrites them into text register. This is the source of the
  old "everything sounds like a call" complaint — never treat transcripts as voice ground truth.
- Ignore for now: `Call Recordings`, `*Recording`, `pipeline-output`, `reports`.

## Pipeline (scripts/)
`clean_bonzo.py` / `clean_transcript.py` → `{file,source,turns}` · `rehydrate.py` (redaction tokens →
realistic surrogates, BEFORE anything sees them — else the model learns to emit `{NAME}` literally) ·
`build_label_tasks.py` (Stream A: real convos → per-agent-turn tasks w/ gold reply) · `gen_scenarios.py`
(Stream B: deterministic screening oracle → stage-coverage CONTEXT blocks) · `gen_maos_scenarios.py`
(emotion-first MAOS scenarios) · `label_with_teacher.py` (teacher infers CONTEXT + writes 3 suggestions,
one anchored on the real reply; `--dry-run` stubs it for local testing) · `label_offline.py` (batched
offline labeling) · `audit_gate.py` (deterministic guardrail drop) · `to_sft.py` (→ chat messages
train/val). `run.sh` runs the whole chain. Helpers: `mine_bonzo.py`, `crosscheck_labels.py`, `stats.py`,
`01_pull_data.sh`.

## Screening oracle (the 4 keys, in order)
`property_use, bankruptcy_past_3yr, foreclosure_past_2yr, late_payments_past_12mo`. Ineligible if any of the
last three is "yes" (reasons: `bankruptcy_within_3yr` / `foreclosure_within_2yr` / `late_mortgage_within_12mo`);
`property_use` is informational, never disqualifying. First-time buyers skip foreclosure & late-mortgage.
Kept in `gen_scenarios.py` — keep it in sync with the prompt.

## Machines
- **Local** (`~/Documents/turn_mom`): build/test scripts on 1–3 samples only (`--dry-run`, `_samples/`). Never bulk-download 21k files here.
- **GH200 box** (`/var/turn-mom`, root@GPU, 45.76.248.215): ~95GB HBM, aarch64/Grace, CUDA 13. All real pull/label runs.
  Serving vLLM caps at ~95 tok/s (~0.28 tasks/s) for teacher labeling — host-bound; a 28k relabel ≈ 28h.

## Gotchas
- **Teacher JSON drift**: `label_with_teacher.py` parses the outermost `{...}` and requests `response_format=json_object`. Always dry-run a `--limit 200` and eyeball voice before a full label run.
- **Rehydrate first**: run `rehydrate.py` before labeling so the teacher never sees `{NAME_GIVEN}` tokens.
- **R2 creds were pasted in chat — rotate them.**

## Conventions
- Redaction placeholders: curly tokens `{NAME_GIVEN} {PHONE_NUMBER} {MONEY} {ORGANIZATION} ...` (already in Clean Redacted).
- `source` tag on every cleaned convo (`bonzo` | `transcript`) — voice sampling prefers `bonzo`.
- CONTEXT block fields (exact order): Stage, Next question key, Answers collected, Ineligibility reason, Client's latest message, Is first message.

---
> Source: [toeeshchaudhary/turn-mom](https://github.com/toeeshchaudhary/turn-mom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
