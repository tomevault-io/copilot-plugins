## pi-peer

> Use `/peer factory status` and `/peer do metrics` before closing substantial peer work. Failed gates should become `/peer factory rework <run-id>` records, not blind retries. Repeated failures should become `/peer context patch --trigger <trigger> --change <change> --metric <metric> --eval <eval-name> --owner <peer-id> --review-date YYYY-MM-DD` proposals followed by `/peer context eval <patch-id> <pass|fail> --eval <eval-name> --evidence <text>`.

# pi-peer agent notes

## Verification-first factory

Use `/peer factory status` and `/peer do metrics` before closing substantial peer work. Failed gates should become `/peer factory rework <run-id>` records, not blind retries. Repeated failures should become `/peer context patch --trigger <trigger> --change <change> --metric <metric> --eval <eval-name> --owner <peer-id> --review-date YYYY-MM-DD` proposals followed by `/peer context eval <patch-id> <pass|fail> --eval <eval-name> --evidence <text>`.

Prefer the mission facade for new substantial work:

```bash
/peer do "Ship a verified improvement to peer coordination" --path src/peers --gate test --gate pack
```

This creates or reuses a peer goal, seeds peer lanes, links a factory run, and prints only the next actions that still need external evidence. If setup is missing, run `/peer setup` and `/peer setup 6`, then repeat the mission command. `/peer accomplish <objective>` is an alias.

Factory workflow quickstart:

```bash
/peer setup id smoke-verifier
/peer setup 6
/peer center
/peer do "Smoke test factory verification" --gate test --gate pack
# run the printed /peer factory plan-review ... command, then run external verification
/peer factory gate <run-id> test pass --evidence "npm test passed"
/peer factory gate <run-id> pack pass --evidence "npm pack --dry-run passed"
/peer factory metrics
/peer center
```

Expected result: the run is `verified`, metrics show one verified run, and `/peer center` stops recommending rework for that run. `/peer do` mission commands create peer/factory records; they do not automatically run shell verification, publish artifacts, or perform remote writes.

## Peer goal-board operating protocol

Before starting substantial peer work, run `/peer center` or inspect the active goal with `/peer get <goal-id>` / `peer_get`. Do not duplicate active claims or work keys. Use read-only claims for research, planning, review, and coordination; claim write work only after naming concrete `--path` values.

For idle-watcher or scout offers: inspect the goal, accept only if the work key is still unclaimed, post concrete evidence (`finding`, `handoff`, `note`, or `vote`), then release the claim. If the claim is duplicate or the board is no longer actionable, stop with a short handoff.

Normal closure requires no unresolved blockers or unsuccessful handoffs, no active/stale claims, no running tasks, all proposals resolved/deferred, at least one current passing vote, and no failed votes. Final peer handoffs must include: `Status`, `Files changed`, `Verification`, `Blockers/risks`, and `Safe for review`.

## Bounded recursive self-improvement

Use `/peer self-improve` when a user wants this repo to improve the peer system through bounded, reviewable experiments.

Common flow:

```bash
/peer self-improve init
/peer self-improve status
/peer self-improve run "Improve peer coordination safety" --loops 10 --path src/peers --eval "npm test"
```

Use dispatch only when explicitly requested or when a bounded peer run is appropriate:

```bash
/peer self-improve run "Improve idle watcher usefulness" \
  --loops 10 \
  --duration 30m \
  --peer worker2,worker3 \
  --dispatch \
  --path src/peers \
  --eval "npm test"
```

What it creates:

- `.pi/self-improve/constitution.md` — user-editable philosophy, non-goals, and promotion rules
- `.pi/self-improve/goals.json` — high-level improvement targets
- `.pi/self-improve/experiments.jsonl` — append-only experiment/run ledger
- a normal peer goal with dependency-gated loop work-items and lane proposals

Safety expectations:

- Runs are bounded; `--loops` is capped at 100.
- Peer dispatch is off by default and requires `--dispatch` plus `--duration`; pass `--peer` to choose peers, or omit it to use active compatible peers.
- Write work still needs explicit paths and should use worktree/branch isolation.
- `--auto-commit` is policy metadata only; it does not automatically publish or bypass review gates.
- Never run `npm publish`, force-push, destructive filesystem/database commands, or weaken tests as part of autonomous self-improvement.
- Promotion requires passing evals, peer review evidence, no unresolved blockers/proposals, and a clear ledger/goal-board handoff.

Implementation references:

- Command parser: `src/peers/command.mjs`
- Runtime wiring: `extensions/pi-peer/index.ts`
- Self-improvement module: `src/peers/self-improve.mjs`
- Tests: `test/peer-self-improve.test.mjs`, `test/peer-command.test.mjs`
- User docs: `README.md` section “Bounded self-improvement runs”

---
> Source: [CryptoLibertus/pi-peer](https://github.com/CryptoLibertus/pi-peer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
