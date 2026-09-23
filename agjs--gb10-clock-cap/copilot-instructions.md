## gb10-clock-cap

> You are running a GPU clock-cap evaluation on one or more inference hosts. This file is the spec. Follow it in order. Do not improvise the measurement method; the thresholds and orderings here exist because the obvious alternatives give wrong answers.

# Agent contract

You are running a GPU clock-cap evaluation on one or more inference hosts. This file is the spec. Follow it in order. Do not improvise the measurement method; the thresholds and orderings here exist because the obvious alternatives give wrong answers.

## One-command path

```bash
./run.sh --hosts "hostA hostB" --api-host hostA --model MODEL_NAME --mode full
```

Emits `results/summary.json`. Read `verdict.recommend` and stop. Everything below is for when you need to deviate.

## Inputs you must resolve before running

| Input | How to get it | Fails if wrong |
|---|---|---|
| `--hosts` | SSH aliases for every node with a GPU under test | Sampler collects nothing |
| `--api-host` | The node serving the OpenAI-compatible API | Benchmarks cannot connect |
| `--model` | `curl $API/v1/models \| jq -r .data[0].id` | Every request 404s |
| `--cap` | Default 2200. Only change with a reason | — |
| `--prefill-tokens` | Mean real prompt size on this system. Default 33000 | Probe does not represent the workload |

Get the mean prompt size from the server, do not guess:

```
vllm:request_prompt_tokens_sum / vllm:request_prompt_tokens_count
```

## Preflight gates

`run.sh --mode preflight` checks all of these and refuses to continue on failure:

1. Every host reachable over SSH without a password prompt.
2. `sudo -n nvidia-smi` works on every host. Clock control needs root.
3. `nvidia-smi -lgc 0,<cap>` applies and `nvidia-smi -rgc` reverts. Some GPUs do not support this. Confirm the reported clock actually changes.
4. API responds and `--model` appears in `/v1/models`.
5. `stdbuf` present on every host.

## Measurement rules

These are not stylistic preferences. Violating any one of them produces a confidently wrong number.

**Normalise throttle time to load seconds, never to uptime.** Read cumulative throttle counters from `nvidia-smi -q -d PERFORMANCE`. Divide by time under load, which for a vLLM server is `vllm:e2e_request_latency_seconds_sum`. On an idle-heavy host the uptime denominator understates throttling by roughly 100x.

**Never conclude anything thermal from an idle reading.** A GB10 idles near 48 C and reaches 90 C under sustained load. Idle temperature carries no information about the question.

**Use a soak, not a request/response benchmark, for temperature.** Gaps between requests let the GPU cool. `ab_test.sh` reliably records zero throttling on hardware that `soak.sh` drives to 90 C minutes later.

**Measure prefill separately from decode.** Decode is memory-bandwidth-bound and barely responds to clock. Prefill is compute-bound and is where the cost lands. A decode-only benchmark will tell you a clock cap is free. It is not free for long-context work.

**Assert the prefill probe stayed cold.** `prefill_probe.py` reports the prefix-cache hit rate it observed. Above 5% the run is invalid; discard it. The probe places its nonce at the start of the prompt for this reason.

**Interleave arms.** Run A B A B, not all-A then all-B. Servers drift; a single split assigns all drift to one arm.

**Include a repeat baseline in any sweep.** Measure stock at both the start and the end. If the two disagree by more than the decode standard error, discard the entire run.

**Do not compare a warm host against a cold one.** All arms cool to the same temperature first.

## Interpreting the numbers

Decode standard deviation runs about 4 tok/s. At n=20 per arm the standard error on a difference is roughly 1.2 tok/s, so any decode gap under about 2.5 tok/s is noise. Do not report it as a result.

Prefill standard deviation runs about 0.2 s. Differences of a few percent there are real.

Weight the two by the host's actual traffic:

```
cost = w_decode * decode_time_delta + w_prefill * prefill_time_delta
```

Derive the weights from the server, not from assumption:

```
w_prefill = vllm:time_to_first_token_seconds_sum / vllm:e2e_request_latency_seconds_sum
w_decode  = 1 - w_prefill
```

## Decision rule

`summary.json` sets `verdict.recommend` to `apply` when all three hold:

- weighted cost is at or under `--max-cost-pct` (default 3.0)
- throttle seconds per load second drop
- peak temperature drops by at least 5 C

Otherwise `revert`. If the thermal gain is nil the answer is `revert` even at zero performance cost, because a persistent config change that buys nothing is still something to maintain.

## Applying

`scripts/install_cap.sh` installs a systemd oneshot. `nvidia-smi -lgc` does not survive reboot and persistence mode does not preserve it.

Order is mandatory: **stop GPU workloads before `systemctl daemon-reload`.** The reload revokes GPU access from already-running containers, and NVML inside them fails until they restart. Set `STOP_CMD` and `START_CMD` in config.env.

Enabled and active is not proof. Reboot a node and re-check before reporting success.

## Rollback

`sudo nvidia-smi -rgc` on every host reverts immediately, no restart required. To remove persistence: `sudo systemctl disable --now gb10-clock-cap.service`.

## Scope limits

Single-stream only. Concurrency changes the compute-to-bandwidth ratio and invalidates these conclusions.

All power figures are the GPU rail from `nvidia-smi`. On GB10 that is a fraction of wall draw (roughly 11 W of 53 W at idle). Do not report the rail as wall power. If you need wall power, read it from a smart plug and say where it came from.

---
> Source: [agjs/gb10-clock-cap](https://github.com/agjs/gb10-clock-cap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
