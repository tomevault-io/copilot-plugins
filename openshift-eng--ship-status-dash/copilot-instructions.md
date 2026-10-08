## testing

> Testing guidelines and constraints for Ship Status Dashboard


* **Never run `make local-e2e` more than once per request.** E2e tests start multiple containers and take several minutes. Run once, capture the output, and read the results. **Do not** re-run e2e just to grep for different things.
* The same applies to `go test ./test/e2e/...` — never run it repeatedly.
* Use `go vet` and `go test` (for unit tests) to validate changes before resorting to a full e2e run.
* E2e tests manage their own PostgreSQL container and dynamically assign ports.

---
> Source: [openshift-eng/ship-status-dash](https://github.com/openshift-eng/ship-status-dash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-08 -->
