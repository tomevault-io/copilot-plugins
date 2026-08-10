## backend-standards

> - Always use **async/await**; never mix callbacks & Promises.

Backend rules:

- Always use **async/await**; never mix callbacks & Promises.  
- For XML parsing, prefer `fast-xml-parser` v5 with `{ignoreAttributes: false,
  allowBooleanAttributes: true}` — it is 2‑3× faster than `xmldom`  [oai_citation_attribution:6‡npm](mdc:https:/www.npmjs.com/package/fast-xml-parser?utm_source=chatgpt.com).  
- Stream large (>10 MB) request bodies with `busboy` and pipe into the parser to
  avoid buffering entire payloads.  
- Centralise error handling in `errorMiddleware.ts` and return JSON of shape
  `{error:{code,message}}`  [oai_citation_attribution:7‡Codez Up](mdc:https:/codezup.com/express-error-handling-best-practices-patterns/?utm_source=chatgpt.com).  
- Wrap heavy XPath evaluation in a **worker_thread**; expose `/evaluate` endpoint
  that streams `{xpath,index,line,col}` ND‑JSON chunks.  
- Log with `pino` at `info` level; rotate daily.  

---
> Source: [mnhlt/Xtracta](https://github.com/mnhlt/Xtracta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
