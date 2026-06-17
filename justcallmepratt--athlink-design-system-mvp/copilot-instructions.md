## athlink-design-system-mvp

> Use this skill to generate well-branded interfaces and assets for Athlink, the international athlete recruiting MVP, either for production or throwaway prototypes/mocks. Contains essential design guidelines, colors, type, fonts, assets, and UI kit components for prototyping.


Read the README.md file within this skill, and explore the other available files.

If creating visual artifacts (slides, mocks, throwaway prototypes, etc), copy assets out and create static HTML files for the user to view. Reference `colors_and_type.css` for tokens (CSS variables), `assets/` for the logo and brand imagery, and the `ui_kits/{recruiter,athlete}/` folders for component/screen patterns.

If working on production code, you can copy assets and read the rules here to become an expert in designing with this brand. The original codebase is `justcallmepratt/Athllinkmvp-DEMO` (Vite + React + TS + Tailwind v4 + shadcn/ui + Lucide).

If the user invokes this skill without any other guidance, ask them what they want to build or design, ask some questions about audience and surface (recruiter desktop vs athlete mobile vs marketing), and act as an expert designer who outputs HTML artifacts _or_ production code, depending on the need.

## Hard rules
- Lucide outline icons, stroke 2 — never emoji as chrome (only flags + sponsor placeholders allowed).
- Light surfaces only — no dark mode in MVP.
- Brand gradient (`#00BCD4 → #2196F3 → #4CAF50`) is reserved for logo, AI Verified pill, primary stat bars. Not card backgrounds.
- Numbers carry the message — lead with stats, use tabular mono for them.
- Hairline `1px #F3F4F6` borders + `shadow-sm` for cards. `rounded-lg` (12px) default.
- Sentence case for actions, Title Case for feature names.

---
> Source: [justcallmepratt/Athlink-Design-System-MVP](https://github.com/justcallmepratt/Athlink-Design-System-MVP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
