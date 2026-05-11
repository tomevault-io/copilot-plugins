## concise-summary

> - name: concise-summary

rules:
  - name: concise-summary
    description: "Avoid long markdown reports after task completion"
    triggers: ["post-task", "summary", "completion"]
    actions:
      - "Do not generate markdown summaries or full reports."
      - "Instead, reply in a concise 'cursor plan' style — 2 to 4 short sentences summarizing what was done or decided."
      - "No markdown formatting (no headings, lists, or code blocks)."
      - "Focus on key actions or outcomes only."

---
> Source: [HyperBDR/devmind](https://github.com/HyperBDR/devmind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-10 -->
