## hoc-claude-ai

> Website tieng Viet huong dan hoc Claude AI qua lo trinh 4 giai doan, 16 bai hoc.

# Website hoc Claude AI

Website tieng Viet huong dan hoc Claude AI qua lo trinh 4 giai doan, 16 bai hoc.

## Tech Stack
- Static HTML/CSS/JS — khong framework, khong build step
- Font: DM Sans + JetBrains Mono (Google Fonts)
- Light/dark mode: CSS `prefers-color-scheme`
- Progress tracking: localStorage

## Project Structure
- `index.html` — Trang roadmap chinh (navigation)
- `lessons/bai-XX-*.html` — 16 trang bai hoc rieng le
- `css/` — base.css (variables, reset), components.css (shared UI), roadmap.css, lesson.css
- `js/` — roadmap.js (accordion + localStorage progress), lesson.js (lesson interactions)
- `assets/images/` — Screenshots va diagrams
- `.claude/skills/frontend-design/` — Skill thiet ke UX/UI tu Anthropic

## Conventions
- HTML: `lang="vi"`, UTF-8
- Lesson filenames: `bai-XX-slug.html` (zero-padded, ASCII slug khong dau)
- CSS custom properties cho colors, spacing, radius (dinh nghia trong base.css)
- Dark mode: light-first, dark override trong `@media` block
- Khong dung external JS dependencies

## Phase Colors
- Phase 1 (purple): progress #7F77DD, badge light #EEEDFE/#3C3489, dark #26215C/#CECBF6
- Phase 2 (green): progress #1D9E75, badge light #E1F5EE/#0F6E56, dark #04342C/#9FE1CB
- Phase 3 (amber): progress #BA7517, badge light #FAEEDA/#854F0B, dark #412402/#FAC775
- Phase 4 (orange): progress #D85A30, badge light #FAECE7/#993C1D, dark #4A1B0C/#F5C4B3

## Commands
- Open locally: `open index.html` hoac Live Server extension trong VS Code

---
> Source: [dangducquan98-code/hoc-claude-ai](https://github.com/dangducquan98-code/hoc-claude-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
