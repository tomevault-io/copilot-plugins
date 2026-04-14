## trustednext

> We are using Next.js 15.3.3 and not Next.js 14.x.x , if there is any issue in the codes , search the web and find documentations related to the issue we are fixing to implement every thing correctly.


We are using Next.js 15.3.3 and not Next.js 14.x.x , if there is any issue in the codes , search the web and find documentations related to the issue we are fixing to implement every thing correctly.

Deep Codebase Understanding (1–5)
Analyze the entire codebase before acting — understand all connected parts and project flow.

Identify shared logic, components, services, and naming conventions across all modules first.

Act only when 95%+ confident in the structure, dependencies, and potential impact.

Ensure full comprehension of the requested task before implementation — ask if unclear.

Scan for existing code that the new feature might reuse — avoid duplication at all costs.

🧠 Smart Structuring & Documentation (6–10)
Start each file with 2–3 lines of comments clearly describing what the file contains and does.

If a file has header comments, update them to reflect your edits and additions.

Strictly enforce a hard limit of 200 lines per file — break down or refactor anything larger.

Use non-similar, descriptive file names that reflect exact contents and purpose.

Enforce a clear folder structure that separates features, components, logic, and assets.

🧹 Clean, Real, and Understandable Code (11–15)
NEVER use mockup data, fake logic, placeholders, or “TODO”s — deliver real, working code only.

NEVER purge/delete any code unless asked explicitly — modify safely or isolate changes.

Ensure every variable, function, and class has a clear, descriptive, and contextual name.

Make the flow of the code self-documenting — avoid confusing short or reused identifiers.

Break logic into small, meaningful pieces that are easy to read, trace, and understand.

🧩 Modularity & Component Logic (16–20)
Each feature must be broken into independent modules, files, or components.

Use reusable services, utilities, or components when logic appears more than once.

Place unrelated concerns (UI, data, logic, auth, routing, etc.) in separate folders.

Make sure each module handles only one purpose — follow strict single responsibility.

When implementing new logic, cross-check all existing modules for reusability first.

🪄 UI Design Rules (21–25)
All UI designs must be modern, clean, futuristic, and visually consistent.

Keep all styling in dedicated, separate files — no inline or scattered styles allowed.

Use reusable UI components for cards, buttons, layouts, or typography.

Avoid repeating UI code — extract visual logic into shared component folders.

Ensure UI files are under 200 lines, and if needed, split into layout, logic, and style parts.

📂 Naming & File Management (26–30)
Use filenames that clearly reflect the contents and avoid similar naming across modules.

All folders should represent clear domain or functional separation.

Update filenames and folder names if their contents or purpose evolve.

Avoid long or abbreviated names that reduce clarity — clarity beats brevity.

Never duplicate files or logic — reuse and centralize when appropriate.

📋 Reporting & Rationale (31–35)
After each modification, explain what caused the issue and how you fixed it.

Describe every decision taken — include context for naming, structure, and logic.

Mention how this change improves readability, reusability, or structure.

Include a clear summary of what was added, changed, or fixed at the end.

Avoid vague reports — always be specific and transparent in responses.
Explain what caused the issue and how you fixed it

🚫 Red Flags to Avoid (36–40)
NEVER create mock logic, filler content, or write “placeholder” anything.

NEVER auto-generate boilerplate that’s not context-aware or directly needed.

NEVER guess the UI or logic — always anchor code in real data and use-cases.

NEVER remove existing logic or structure unless you fully understand and justify the change.

NEVER create bloated files — enforce strict modularity and separation as a core rule.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/www-e) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
