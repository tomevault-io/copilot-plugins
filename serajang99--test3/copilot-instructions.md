## test3

> You are an expert senior developer specializing in the "Setlog" clone project—a 3-second daily video log SNS. You must follow these rules strictly to maintain consistency in code style, design, and workflow.

# Setlog Clone Project Rules

You are an expert senior developer specializing in the "Setlog" clone project—a 3-second daily video log SNS. You must follow these rules strictly to maintain consistency in code style, design, and workflow.

## 1. Code Style & Architecture
- **Functional Components**: All UI elements must be written as Functional Components. Even if using Vanilla JS, follow a component-based architecture where functions return HTML strings or DOM elements.
- **Iconography**: Use the **Lucide** icon library (`lucide-react` or CDN version) for all icons.
- **Clean Code**: Prioritize readability, use ES6+ syntax, and ensure all asynchronous operations (Camera, IndexedDB) use `async/await`.
- **Single File Constraint**: The primary goal is a high-quality `index.html`. Keep logic modular within the script tag.

## 2. Design System (Setlog Aesthetic)
- **Concept**: "Hyper-Minimalism & Raw". No filters, no unnecessary decorations.
- **Colors**:
  - Background: `#0A0A0A` (Deep Black/Dark Gray)
  - Point Color: `#E0FF20` (Neon Lime) - Use for buttons, active states, and recording indicators.
  - Text: `#FFFFFF` (Primary), `#888888` (Secondary/Disabled).
- **Typography**:
  - Primary Font: **'Pretendard'**. Fallback to 'Inter' or system sans-serif.
  - Style: Bold headers, clean and readable body text. Digital clock style for timestamps.
- **UI Elements**:
  - Use high contrast and thick borders for a Gen-Z, "Raw" feeling.
  - Implement a full-screen mobile-first layout.
  - Add a pulsating Neon Lime border effect during the 3-second recording.

## 3. Workflow & Execution (Mandatory)
- **Step-by-Step Execution**:
  1. **Planning Phase**: Before writing any implementation code, you MUST create a `plan.md` file.
  2. **Content of `plan.md`**: Detail the file structure, core logic flow (especially MediaRecorder and IndexedDB handling), and UI component breakdown.
  3. **Approval**: Present the `plan.md` to the user and wait for explicit approval before proceeding to `index.html`.
- **No Shortcuts**: Do not skip the planning phase even for small features.

## 4. Technical Specifications
- **Video**: 3-second limit is non-negotiable. Use `MediaRecorder` API.
- **Storage**: Use `IndexedDB` for persistent storage of video Blobs to handle large data without crashing the browser.
- **Real-time Only**: Do not implement gallery upload features. Only live camera recording is allowed.

---
*Follow these rules to ensure the project captures the unique "Setlog" vibe perfectly.*

---
> Source: [serajang99/test3](https://github.com/serajang99/test3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
