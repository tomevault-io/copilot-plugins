## portfolio

> - **Framework**: React (v19.x)

# Project Context & Rules

## Verified Tech Stack

- **Framework**: React (v19.x)
- **Build Tool**: Vite (v7.x)
- **Language**: JavaScript (JSX) - *Sẽ migrate sang TypeScript*
- **Styling**: Tailwind CSS (v4.x)
- **Animation**: Framer Motion (v12.x)
- **i18n**: i18next, react-i18next
- **Icons**: Lucide React
- **Backend**: Firebase (Realtime Database)

## Rule Hub

Dự án này sử dụng modular approach cho AI rules. Vui lòng đọc documentation cụ thể trong `src/docs/rules/` dựa trên context của task.

### Core Rules

- [Architecture & Directory Structure](src/docs/rules/architecture.md)
- [Coding Standards (JSX, Hooks)](src/docs/rules/coding-standards.md)
- [Implementation Flow (React)](src/docs/rules/implementation-flow.md)
- [Common Libraries](src/docs/rules/common-libraries.md)
- [Senior Engineering Mindset (UX/Perf)](src/docs/rules/senior-mindset.md)
- [Commit Convention](src/docs/rules/commit-convention.md)

## Critical Instructions for AI

- **Act as a Senior Engineer**: Áp dụng principles trong `senior-mindset.md` (Performance, UX, Accessibility).
- **No Reinventing**: Check `common-libraries.md` trước khi tạo helpers mới hoặc cài packages.
- **Read Context First**: Khi bắt đầu task mới, check `src/docs/rules/` folder để tìm guidelines phù hợp.
- **Strict Adherence**: Follow patterns defined trong `architecture.md` khi thêm components mới.
- **i18n First**: Tất cả text phải đi qua i18next, không hardcode.
- **Tailwind First**: Sử dụng Tailwind CSS cho styling, tránh custom CSS trừ khi thực sự cần.
- **Framer Motion**: Sử dụng cho animations, follow performance best practices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nguyenduydan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
