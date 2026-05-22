## tools

> | Tool                 | Purpose                                             | Key Usage Rules |

# 🚀 Allowed Tools & Usage Policy

| Tool                 | Purpose                                             | Key Usage Rules |
|----------------------|-----------------------------------------------------|-----------------|
| **search_codebase**  | Vector RAG search across the project codebase (incl. `.knowlenge`). | • Ideal for “where is an example?” queries.<br>• Combine with `grepped_codebase` to narrow results. |
| **grepped_codebase** | Keyword / RegExp search inside source files.        | • Fast exact‑string matching.<br>• Best used after `search_codebase` for precision. |
| **search_web**       | Internet search (Google‑style).                     | • For external libs, docs, blog posts.<br>• May chain with `curl https://…` to fetch a page. |
| **git clone <repo>** | Clone a third‑party repository.                      | • Clone **only** into `.knowlenge/`.<br>• Rename script files (`.cs`, etc.) to `.cstxt` to keep compilers silent. |

## Global Restrictions
1. All cloned repos and scratch files must live in `.knowlenge/`.
2. After cloning, rename scriptable files to their `*.txt` equivalents so build tools ignore them.
3. Recommended search flow:  
   `search_codebase` → refine with `grepped_codebase` → fall back to `search_web` if needed.

---
> Source: [publicrust/rust-template](https://github.com/publicrust/rust-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
