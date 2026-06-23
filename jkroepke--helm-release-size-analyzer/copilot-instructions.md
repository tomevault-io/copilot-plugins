## helm-release-size-analyzer

> The following guidelines apply to all files in this repository.

# Instructions for AI Agents

The following guidelines apply to all files in this repository.

Before making changes, read [`CONTRIBUTING.md`](CONTRIBUTING.md) and
[`DEVELOPER.md`](DEVELOPER.md). Follow the contribution and validation workflow
in `CONTRIBUTING.md`; do not duplicate it here.

Preserve unrelated worktree changes. Verify an uncertain [Helm](https://github.com/helm/helm), Kubernetes, [Cobra](https://github.com/spf13/cobra)
or [Viper](https://github.com/spf13/viper) behavior against a current documentation rather than relying on memory.

All documentation changes must comply with
[textlint-rule-terminology](https://github.com/sapegin/textlint-rule-terminology)
and its
[terminology ruleset](https://github.com/sapegin/textlint-rule-terminology/blob/master/terms.jsonc).
A mandatory rule, because CI checks it.

Keep `README.md` and `DEVELOPER.md` aligned with verified behavior. User
documentation must cover installation, commands, flags, examples, output
definitions, in-memory limitations, and the no-cluster-access guarantee.

---
> Source: [jkroepke/helm-release-size-analyzer](https://github.com/jkroepke/helm-release-size-analyzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
