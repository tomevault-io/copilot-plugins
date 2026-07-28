## continuous-clearing-auction

> Local test validation workflow for this repository


## Test Validation

- When running tests locally to validate a change during agent work, prefer a low fuzz-run count or exclude the auction invariant tests unless the user explicitly asks for a full suite.
- The default full `forge test` run can take a long time in this repository, so use a bounded command such as:

```sh
forge test --fuzz-runs 64 --no-match-contract AuctionInvariantTest
```

---
> Source: [Uniswap/continuous-clearing-auction](https://github.com/Uniswap/continuous-clearing-auction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
