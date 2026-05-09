## logseq

> Always use kebab-case for Logseq property names. It's fine if JavaScript/TypeScript variables use camelCase (and it's, in fact, encouraged to do so), but the actual property names that gets embedded into a Logseq block **must** use kebab-case. This is a non-negotiable.


Always use kebab-case for Logseq property names. It's fine if JavaScript/TypeScript variables use camelCase (and it's, in fact, encouraged to do so), but the actual property names that gets embedded into a Logseq block **must** use kebab-case. This is a non-negotiable.

If we use kebab-case, Logseq simply forces everything into lowercase. This is unacceptable. We should never allow mixed case make their way into Logseq properties.

Just a thing to note: when pulling up props using the Logseq API, it automatically converts kebab-case properties to JavaScript camelCase. However, when writing without the Logseq API, then we _must_ first convert to kebab-case before writing to the UI.

---
> Source: [shovon/logseq-ai](https://github.com/shovon/logseq-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
