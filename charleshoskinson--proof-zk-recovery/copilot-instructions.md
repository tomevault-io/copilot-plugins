## proof-zk-recovery

> > Why run GuardVibe when the code is already carefully reviewed? It is the deterministic, whole-repo, author-independent check with daily CVE intel — it knows vulnerabilities published *after* the model's training cutoff and catches what a single-file view misses. Treat its findings as ground truth to triage, not a second opinion to dismiss.

# Project Guidelines

## Security workflow (GuardVibe MCP)

> Why run GuardVibe when the code is already carefully reviewed? It is the deterministic, whole-repo, author-independent check with daily CVE intel — it knows vulnerabilities published *after* the model's training cutoff and catches what a single-file view misses. Treat its findings as ground truth to triage, not a second opinion to dismiss.

This is a zero-knowledge Cardano recovery project: a gnark circuit (Go), a Plutus V3 validator (Haskell), Lean proofs, a Go prover/vending stack, and a browser-based victim redeemer web app (`product/`, `proto/browser/`, `proto/site/`). GuardVibe's web/JS rules apply mainly to that browser surface; its secret-scanning, supply-chain, and CI rules apply repo-wide.

### Mandatory checks
- AFTER editing anything under `product/`, `proto/browser/`, or `proto/site/` (the browser prover / redeemer web app): use the guardvibe `scan_file` MCP tool on the file — this is the main XSS/SSRF/CORS surface.
- AFTER touching a GitHub Actions workflow, a `package.json`, or a shell install script under `proto/env/`: `scan_file` it (supply-chain / CI hardening rules).
- BEFORE committing: use the guardvibe `scan_staged` MCP tool (diff-aware — flags only newly-introduced issues).
- When GuardVibe reports HIGH or CRITICAL on code you just wrote: fix before moving on. For pre-existing findings in prototype/strawman code, triage rather than reflexively fix — say so explicitly.
- BEFORE opening a PR: review the diff with `scan_changed_files`.

### Scan visibility — always report GuardVibe results
- AFTER every scan: tell the user the GuardVibe result, even when clean.
- Issues: "GuardVibe: [count] issues ([severity]) in [file] — triaging/fixing."
- Clean: "GuardVibe: scanned [file] — no issues."
- Never silently consume scan results.

### Secure-code rules that have real surface here
- Browser app: never assign user/chain-derived data to `innerHTML` — use `textContent`, or sanitize with DOMPurify. (The redeemer renders eligibility/claim data; this is the live XSS surface.)
- Go `proto/browser/server.go` and workers: no CORS `*` wildcard on anything reachable off-localhost; validate any user/param-controlled URL before `fetch`/HTTP (SSRF — block private ranges).
- Secrets: never hardcode keys; keep `.env*` out of the repo. The **only** committed key material is the deliberately-unignored zero-value Preview testnet wallet (`proto/wallet/payment.*`) — never add mainnet or economically-meaningful keys.
- Shell install scripts (`proto/env/`): prefer download-then-inspect over pipe-to-shell; never inline sudo passwords.
- CI: pin third-party actions, set `persist-credentials: false` on `actions/checkout`.

### When in doubt
- `explain_remediation` with the rule ID for fix guidance; `check_code` to verify a snippet before applying.
- Not everything GuardVibe flags in strawman/prototype code (`product/redeemer.html`, `proto/browser/`) needs an immediate fix — but every finding should be consciously triaged, not ignored.

---
> Source: [CharlesHoskinson/proof-zk-recovery](https://github.com/CharlesHoskinson/proof-zk-recovery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
