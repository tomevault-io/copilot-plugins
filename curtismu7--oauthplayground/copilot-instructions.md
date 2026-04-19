## oauthplayground

> - Prefer **flow-specific façades** wrapping shared libs.


## 1️⃣ Project Rules

### **Isolation by Design**
- Prefer **flow-specific façades** wrapping shared libs.  
- Shared logic lives behind interfaces; swapping implementations should not affect other flows.  
- If a change could alter behavior for another flow, **fork & version**:---
alwaysApply: true
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/curtismu7) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
