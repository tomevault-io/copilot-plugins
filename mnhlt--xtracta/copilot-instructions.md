## frontend-standards

> When editing React components:

When editing React components:

- Use **function components** with hooks; no class components  [oai_citation_attribution:2‡Codez Up](mdc:https:/codezup.com/build-live-code-editor-react-monaco-editor/?utm_source=chatgpt.com).  
- Keep each file under **300 lines**; break large views into child components.  
- Styling: **Tailwind utility classes** plus shadcn/ui primitives; never inline CSS.  
- Animate entry/exit with **Framer Motion** variants when element visibility changes.  
- State: central store via **Zustand**; subscribe selectors to avoid over‑render  [oai_citation_attribution:3‡Rui Tao's Portfolio](mdc:https:/ruit.me/blog/mastering-zustand-state-management-in-react?utm_source=chatgpt.com).  
- Editor integration: leverage `monaco.editor.deltaDecorations` for XPath match
  highlighting; throttle updates with `requestAnimationFrame`.  
- For Web Worker <‑> UI messaging, post structured JSON `{xpathResults: ...}`.  
- Use **WASM‑compiled FontoxPath** in the worker and ensure it lazy‑loads to keep
  main bundle ≤ 250 kB gz  [oai_citation_attribution:4‡Stack Overflow](mdc:https:/stackoverflow.com/questions/47083951/how-to-use-webassembly-wasm-code-in-a-web-worker?utm_source=chatgpt.com) [oai_citation_attribution:5‡Cursor](mdc:https:/docs.cursor.com/context/rules?utm_source=chatgpt.com).  

---
> Source: [mnhlt/Xtracta](https://github.com/mnhlt/Xtracta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
