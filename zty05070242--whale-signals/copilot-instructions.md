## whale-signals

> On-chain Ethereum whale tracking + NLP sentiment analysis to model short-term

# Claude Code Instructions for `whale_signals`

## Project Context

On-chain Ethereum whale tracking + NLP sentiment analysis to model short-term
price impact. Research question: do large transactions (>$1M) predict ETH price
movements, and does sentiment moderate this?

See `docs/project_arc.md` for the full phase roadmap, current status, and
session handoff notes. Read it at the start of every new session.

Fred is a Tonmeister (audio/DSP) student applying to MSc FinTech programmes
(HKU MFFinTech). The project must show genuine blockchain engagement, rigorous
methodology, and honest reporting — not just "AI applied to crypto prices".

## Communication Style

- Be direct and honest. If a design choice is questionable, say so before coding.
- Explain the why, not just the what.
- British English in comments and documentation.
- No emojis.

## Code Standards

- **Inline comments on every non-trivial library call.** Fred is self-taught in
  pandas/sklearn/web3.py. Do not assume familiarity.
- **Type hints** on all functions. **Docstrings** on all public functions.
- **Modular design.** Each component independent and testable.
- **No hardcoded paths.** Use `config.py` or environment variables.
- **No look-ahead bias.** If future data is used, flag it loudly and mark the
  function `# ACADEMIC USE ONLY`.
- **Test every data transformation** — edge cases, empty data, timezone issues.
- **Explain before coding.** Stop and teach concepts Fred hasn't seen before.
- **Build incrementally.** No untested features.

## Domain Notes

- Exchange wallets are publicly labelled (Etherscan). This is how whale-watching
  firms classify movements.
- Whale-to-exchange does not guarantee selling. Output probabilities, not certainties.
- High gas price = urgency signal — often a stronger directional indicator.
- MEV bots appear as "whale" transactions but carry no directional signal.
- The 24h prediction horizon is arbitrary. Test multiple (1h, 6h, 24h, 72h).

## Anti-Patterns

- Don't generate large code blocks without explaining reasoning first.
- Don't use unfamiliar libraries without justification.
- Don't fake results. Chance-level performance is an honest finding.
- Don't paper over look-ahead bias.
- Don't treat this as generic "sentiment + price prediction". The on-chain
  transaction analysis is the differentiator.

## Session Protocol

1. Read `docs/project_arc.md` for current phase and context.
2. Explain what you're about to do and why.
3. Show code with inline comments.
4. Note caveats, limitations, things to verify.
5. Suggest the next logical step.

---
> Source: [zty05070242/whale-signals](https://github.com/zty05070242/whale-signals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
