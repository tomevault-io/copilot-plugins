## testing-cheatsheet

> Testing checklist & snippets


Quick reference for high‑impact tests:

1. **XPath parser fuzz**: generate 1 000 random yet valid XPath strings and assert no throw.  
2. **DOM highlight delta**: diff decoration IDs before & after evaluate.  
3. **Backend load**: k6 script `load-test.js` targeting 50 rps, 10 MB docs.  
4. **Lighthouse performance**: budget ‑‑performance ≥ 90 for `/#/` route.  

---
> Source: [mnhlt/Xtracta](https://github.com/mnhlt/Xtracta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
