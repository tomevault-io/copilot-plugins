## java-spiffe

> Review documentation and examples for correctness against the current code and SPIFFE semantics.


# Documentation And Examples

Review documentation and examples for correctness against the current code and SPIFFE semantics.

Check:

- public API names, package names, artifact names, and module names match the implementation
- examples preserve security-sensitive defaults and do not weaken verification, trust-domain checks, or provider behavior
- commands, Gradle snippets, environment variables, socket paths, keystore/truststore paths, and passwords are clearly scoped to examples or local testing
- for SPIFFE semantics, prefer the official standards at `https://github.com/spiffe/spiffe/tree/main/standards`; do not rewrite normative language casually
- README or example changes mention compatibility, configuration, or behavior changes when the corresponding code changed

Flag docs that teach unsafe patterns, expose secrets, imply production defaults for test-only settings, or drift from the public API.

---
> Source: [spiffe/java-spiffe](https://github.com/spiffe/java-spiffe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-23 -->
