## no-em-dashes-in-ad-copy

> Operator-mandated hard rule - never use em dashes in ad copy


# Never use em dashes in ad copy

The operator has permanently banned em dashes (—) from all ad copy written in
this repo: primary text, bodies, titles, headlines, descriptions, hooks, CTAs,
copy.json files, and any draft or variant of Meta/Facebook/Instagram copy.

- No exceptions, ever. Do not ask; do not use them "stylistically."
- Rewrite around them: use a period, comma, colon, or split the sentence.
- Before delivering any copy, confirm zero `—` characters are present.
- `skills/human-ad-copy/scripts/validate_copy.py` marks em dashes as HARD
  failures. Always run it on new copy and fix every em-dash finding.

```text
❌ Automation changed everything — no more manual builds.
✅ Automation changed everything. No more manual builds.
```

---
> Source: [krusemediallc/cursor-ad-agent](https://github.com/krusemediallc/cursor-ad-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
