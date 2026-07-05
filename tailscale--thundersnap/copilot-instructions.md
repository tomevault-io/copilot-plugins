## thundersnap

> - Always run "make test" after making changes. "make test" also enforces gofmt;

# Claude Code Project Notes

- Always run "make test" after making changes. "make test" also enforces gofmt;
  run "gofmt -w ." to fix formatting before committing.
- To build: "make binaries" (puts them in bin/) or "make ts" (just the ts binary)
- e2e tests MUST NEVER SKIP. "make e2e" must either fully pass every validation
  or fail. Never add t.Skip/t.Skipf/t.SkipNow to e2e tests; if a precondition
  (root, btrfs, VM deps) is missing, that is a misconfigured environment and the
  test must fail (t.Fatal), not skip. The e2e package is built with the "e2e"
  build tag so it is excluded from "make test" and only run via "make e2e".
- e2e tests run simplest-to-hardest in tiers (see e2e/main_test.go TestMain); if
  an early tier fails, later tiers are skipped so you debug the root cause first.
- To run a single e2e test, use E2E_ARGS with -test.run and log to a file:
  `make e2e E2E_ARGS="-test.run=TestSSHContainerBasic" 2>&1 | tee e2e.log`
  Always log output to a file (e2e tests are verbose). Tests typically complete
  in ~30s; use a 1-2 minute timeout when waiting.
- This project workspace may be using the 'jj' tool instead of git. Always
  check for jj first. If using jj, when making a fix, always `jj describe` it
  when done and then `jj new` so you don't accidentally mix changes
  together. Be careful that `jj log` shows not just this branch, but other
  branches mixed into the hierarchy!

---
> Source: [tailscale/thundersnap](https://github.com/tailscale/thundersnap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
