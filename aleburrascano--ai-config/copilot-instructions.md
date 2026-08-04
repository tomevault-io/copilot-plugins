## ai-config

> `okf/` is the curated knowledge bundle (OKF format: markdown concepts, YAML frontmatter, one concept per file). Start at `okf/index.md` and descend only into the branch you need — before exploring an unfamiliar module, read its concept doc first. A pre-commit hook blocks commits that change a concept's `resource:` files without updating the concept.

## Knowledge base

`okf/` is the curated knowledge bundle (OKF format: markdown concepts, YAML frontmatter, one concept per file). Start at `okf/index.md` and descend only into the branch you need — before exploring an unfamiliar module, read its concept doc first. A pre-commit hook blocks commits that change a concept's `resource:` files without updating the concept.

## Patterns

`~/.claude/lexicon/` is the authoritative pattern reference. Manifests are **never auto-loaded** — Read the language manifest (`MANIFEST-go.md` / `MANIFEST-ts.md`) before proposing or rejecting any abstraction, and check `INDEX.md` for cross-cutting manifests (caching, event-driven, observability…) when the work touches those domains. Full entries at `~/.claude/lexicon/site/{path}/index.html` — Grep for `Avoid|Cost` and quote the cost line when tradeoffs matter. When proposing an abstraction: name its manifest pattern — or "no pattern — direct code" **plus the closest manifest entry and why it loses** (an unchecked "no pattern" is an assertion, not a verdict) — name the concrete second implementation ("flexibility" isn't one), and honor its _Cost:_ line — no cost line means unproven, not free. One constraint outranks any pattern: imports point one direction, the object graph is wired explicitly in the composition root, behavior lives with its data.

---
> Source: [aleburrascano/ai-config](https://github.com/aleburrascano/ai-config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
