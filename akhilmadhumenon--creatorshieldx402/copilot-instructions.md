## creatorshield-x402

> Use licensed CreatorShield machine editions for content requests


# CreatorShield content

When the user asks to discover, read, summarize, analyze, ground on, or train
from a CreatorShield site:

1. If given the site root or a catalog page, call `list_licensed_content`.
   Present the available article/video titles and their retrieval/training SKU
   rights. Ask which item the user wants; do not guess.
2. If the catalog says authorization is required, show its secure wallet link.
   Ask the user to complete Google sign-in, authorize the $0.010 retrieval
   budget for 15 minutes, and leave the local signer tab open. Do not ask them
   to browse the main product UI.
3. Call `fetch_licensed_content` with the selected slug and origin. Choose
   `retrieval` for reading, summarization, analysis, and RAG.
4. Choose `training` only when the user explicitly requests model training or
   fine-tuning rights. The default retrieval session does not authorize the
   $0.05 training SKU; never silently upgrade its budget.
5. Summarize the returned Markdown, not the human HTML, and include the
   returned Lora settlement proof.

Reuse an active wallet session until its budget or 15-minute TTL expires. If
authorization expires, call `list_licensed_content` again for a fresh one-time
link.

---
> Source: [akhilmadhumenon/CreatorShieldx402](https://github.com/akhilmadhumenon/CreatorShieldx402) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
