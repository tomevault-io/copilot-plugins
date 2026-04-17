## cognit-thinking-book

> This project is a hybrid of a **book-writing project** and its supporting **web workspace**. The book, *Cognit Thinking*, is a cognitive upgrade manual for the AI era, focusing on AI mindset, system orchestration, and human-AI collaboration.

# Project Overview: 《智元思维》 (Cognit Thinking)

This project is a hybrid of a **book-writing project** and its supporting **web workspace**. The book, *Cognit Thinking*, is a cognitive upgrade manual for the AI era, focusing on AI mindset, system orchestration, and human-AI collaboration.

## 📖 Book Project Structure

The book is organized into four main parts, with a total of 18 chapters plus supplementary materials.

### Directory Organization
- **`第一部-觉醒` to `第四部-进化`**: The core content of the book. Each chapter resides in its own directory (e.g., `第01章-AI时代的认知陷阱/`).
- **`序言/`, `尾声/`, `附录/`**: Supporting book sections.
- **`设计系统/`**: Contains architectural birds-eye views, writing style guides, and production tracking.
- **`assets/figures/`**: SVG illustrations used throughout the book.
- **`读者测评/`**: Assessment systems for readers to evaluate their "Cognit Thinking" level.

### Chapter File Conventions
Each chapter directory typically contains:
- `大纲.md`: Chapter structure and section lists.
- `深度展开.md`: Detailed framework, skeletal content, and case structures.
- `AI协作实战.md`: Specific prompt engineering systems related to the chapter.
- `案例集.md`: Real-world scenarios and case studies.
- `正文初稿.md`: Full text drafts (primarily for early chapters).

### Writing Style Guidelines
- **Tones**: Intellectual equality, specific over abstract, opinionated but grounded, and transparent about uncertainty.
- **Banned Buzzwords**: Avoid empty corporate jargon like "赋能" (empower), "颠覆" (disrupt), "底层逻辑" (underlying logic), etc.
- **Focus**: Practical execution over mere theory.

---

## 💻 Cognit Workspace (Code Project)

Located in the `cognit-workspace/` directory, this is a web application that implements the methodologies described in the book.

### Technologies
- **Framework**: Next.js 16.2.3 (App Router)
- **Library**: React 19.2.4
- **Styling**: Tailwind CSS 4.x (with @tailwindcss/typography)
- **Language**: TypeScript 5.x
- **Content**: React-markdown and Remark-GFM for rendering book content.

### Key Commands
Run these commands within the `cognit-workspace/` directory:
- **Development**: `npm run dev`
- **Build**: `npm run build`
- **Start**: `npm run start`
- **Lint**: `npm run lint`

### Architecture
- `src/`: Source code for the Next.js application.
- `public/`: Static assets for the web workspace.
- `eslint.config.mjs` & `tsconfig.json`: Project configuration.

---

## 🛠 Usage and Contribution

- **Writing**: Follow the guidelines in `设计系统/写作风格规范.md`. Ensure new chapters follow the established file naming conventions.
- **Figures**: Place new SVG diagrams in `assets/figures/` following the naming pattern (e.g., `S-XX.svg` for core concepts, `C-XX.svg` for cases).
- **Tooling**: When updating the workspace, ensure compatibility with React 19 and Tailwind 4.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/huxiangang-art) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
