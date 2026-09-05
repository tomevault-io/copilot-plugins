## paperastra

> Read `AGENT_GUIDE.md` before changing or using this repository.

# PaperASTRA agent contract

Read `AGENT_GUIDE.md` before changing or using this repository.

PaperASTRA is a focused Evidence-to-Cinema tool: it accepts a paper URL or PDF,
extracts paper-native evidence, writes traceable artifacts, and renders a
30-second scientific promo through `PaperAstraPromo`.

Keep the public surface small:

```powershell
paperastra run <paper-url-or-pdf>
```

The module entry point is also available as `python -m paperastra run ...`.

Preserve the product language:

- source paper -> evidence ledger -> narrative spine -> shotbook -> Style Card
  -> PaperAstraPromo render;
- six purposes only: `thesis_open`, `problem_pressure`, `method_anatomy`,
  `proof_frame`, `implication_landing`, `source_lockup`;
- current Style Cards only: `obsidian-lab`, `swiss-signal`, `prism-field`;
- every major claim must remain traceable to page numbers, source sentences,
  claim IDs, and figure IDs where available.

Do not broaden the repository into a general video studio, hide renderer
branches behind Style Cards, or weaken the evidence contract. Generated
projects, downloaded papers, voice models, dependency
folders, and rendered outputs do not belong in the source tree.

---
> Source: [Nova-chen151/PaperASTRA](https://github.com/Nova-chen151/PaperASTRA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
