## f1-strategy-copilot

> A tool-calling LLM agent that turns a deterministic F1 race-strategy model's

# F1 strategy copilot — build brief for Claude Code

## What this project is
A tool-calling LLM agent that turns a deterministic F1 race-strategy model's
output into grounded natural-language recommendations, evaluated against
real historical races on two metrics: strategy regret (seconds gained/lost
vs. what actually happened) and citation hallucination rate (does every
claim in the LLM's output trace back to a real tool result).

## Non-negotiable rules for how you (Claude Code) work through this
1. Work through the phases below IN ORDER. Do not start phase N+1 until
   phase N's acceptance check passes and has been shown to the user.
2. After each phase, STOP and report: what was built, what the acceptance
   check showed (paste real numbers/output, not a description of what it
   would show), and any assumption you made that the user should confirm.
3. Do not fabricate example output, sample data, or "for demonstration"
   numbers anywhere. If real data isn't available yet (e.g. ingestion
   hasn't run), say so explicitly rather than filling in a plausible-looking
   placeholder — placeholders that look real are the single most common way
   this kind of project quietly ships broken.
4. `data/ingest_fastf1.py` already exists (provided) — don't rewrite it,
   extend it if a phase needs new fields.

---

## Phase 0 — environment check
- Confirm `fastf1`, `pandas`, `pyarrow`, `scikit-learn`, `anthropic` import
  cleanly.
- Run `data/ingest_fastf1.py --year 2023 --gp Bahrain --session R` and
  confirm real rows land in `data/processed/`. If it returns zero laps,
  STOP — do not proceed until this works, since every downstream phase
  depends on real data existing.
- Acceptance check: print row counts for laps, weather, telemetry files.

## Phase 1 — degradation + pace model (`engine/degradation.py`, `engine/pace.py`)
- Fit a per-compound lap-time-vs-tyre-age model using ONLY laps flagged
  `clean_air == True` from the ingestion output. Using unfiltered laps
  will produce a model that looks fit but is actually capturing SC/traffic
  noise — this is the most common silent failure mode in this phase.
- Report R² per compound per circuit fit, not just for a single race.
- Acceptance check: R² values must be printed and must be > 0.4 to
  proceed (below that, the model isn't usable for downstream strategy
  calls — flag it and ask the user rather than continuing anyway).

## Phase 2 — strategy calc (`engine/strategy_calc.py`)
- `undercut_delta(driver, lap)`: expected time gain from pitting one lap
  before a rival, using the degradation model from phase 1.
- `pit_loss_window(circuit)`: static pit-lane loss estimate + dynamic
  discount if a safety car is active this lap (from `TrackStatus`).
- Acceptance check: run against 3 real historical laps where you know
  the actual strategy call from race results, and show the calculated
  delta alongside what actually happened. Do not just show that the
  function runs — show it runs on real inputs with a sensible number.

## Phase 3 — race simulator + regret metric (`engine/race_sim.py`, `eval/backtest.py`)
- Given a full race's lap data and a candidate strategy (pit lap +
  compound choice), simulate total race time using the phase 1 model.
- For each backtested race: compute simulated time under (a) the model's
  recommended strategy and (b) the driver's actual real strategy. Regret
  = (a) − (b) in seconds. Negative regret = model would have been faster.
- Acceptance check: a table of regret per race across the 8-race backtest
  set, plus mean/median. This number is the project's headline metric —
  do not let this phase pass with fewer than 5 of the 8 races actually
  computing successfully.

## Phase 4 — agent (`agent/tools.py`, `agent/prompts.py`, `agent/agent.py`)
- Tool functions wrap phase 1-3 functions directly — no new logic here,
  just JSON-serializable wrappers.
- System prompt must explicitly require: every strategic claim in the
  output cites the tool call and field it came from (e.g. "per
  undercut_delta: 1.4s/lap"). No tool result, no claim.
- Log every (tool call, tool result, generated text) triple to
  `agent/logs/` — this is required input for phase 5, not optional.
- Acceptance check: run the agent on 3 real race situations from the
  backtest set, show full transcripts including tool calls.

## Phase 5 — hallucination check (`eval/hallucination_check.py`)
- Parse the logged transcripts from phase 4. For each generated claim
  with a citation, verify the cited number actually appears in the
  corresponding tool result. Report hallucination rate = (uncited or
  mismatched claims) / (total claims).
- Acceptance check: the actual rate on real transcripts, plus at least
  one concrete example of any hallucination found (or explicit
  confirmation none were found in the sample).

## Phase 6 — dashboard (`dashboard/app.py`, Streamlit)
- Only start this after phases 0-5 pass. Panels: race replay controls,
  gap-to-leader chart, degradation chart, strategy recommendation card
  (with citations visible), and a metrics strip showing the real regret
  and hallucination numbers from phases 3 and 5 — not placeholder values.
- Circuit map (canvas-based car positions) is a stretch addition after
  the above is working, not a prerequisite for a working demo.

## Definition of done
README.md front-loads: mean regret across backtest set, hallucination
rate, and a link/gif of the running dashboard. If any of these three
things doesn't exist yet, the project isn't done, regardless of how much
UI has been built.

---
> Source: [RuthiKode/f1-strategy-copilot](https://github.com/RuthiKode/f1-strategy-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
