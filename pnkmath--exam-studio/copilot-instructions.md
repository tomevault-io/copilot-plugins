## exam-studio

> Exam Studio is a workflow platform for creating mathematical exam papers in the HWPX (Hancom Word Processor Open XML) format. It combines a Next.js web interface with an agentic pipeline (Claude/Gemini) and custom Python document manipulation tools.

# Exam Studio: Math Exam Builder

Exam Studio is a workflow platform for creating mathematical exam papers in the HWPX (Hancom Word Processor Open XML) format. It combines a Next.js web interface with an agentic pipeline (Claude/Gemini) and custom Python document manipulation tools.

## Project Overview

- **Core Purpose:** Automate the extraction of math problems from PDFs, generate solutions/explanations, process geometric figures, and assemble high-quality HWPX documents.
- **Main Technologies:**
    - **Frontend/Orchestrator:** Next.js (TypeScript), React, Tailwind CSS, Zustand.
    - **AI providers (stage 실행):** Codex CLI (권장) / Claude Code CLI / Claude·OpenAI SDK / DeepSeek — `/settings` 에서 지정. Gemini (그림 처리).
    - **Document Engine:** Custom Python scripts (`build_hwpx.py`, `fix_namespaces.py`, `validate.py`) for ZIP-level XML manipulation of HWPX files.
    - **PDF Processing:** PyMuPDF (`fitz`) for image extraction and layout analysis.

## Directory Structure

```text
/
├── .claude/               # Agent definitions and skills
│   ├── agents/            # Specialized Claude agents (extractor, builder, etc.)
│   └── skills/            # Shared capabilities (hwp-equation, exam-create, exam-crop)
├── docs/                  # Architecture and guidelines docs
├── inputs/                # Source PDFs and HWPX templates
├── outputs/               # Generated HWPX files and images
├── studio/                # Main Next.js application
│   ├── app/               # Routes and API endpoints
│   ├── components/        # React components (UI, pipeline, cropper)
│   ├── lib/               # Shared logic (Claude integration, state, utils)
│   └── server/            # Custom SSE server and background stages
├── workspaces/            # Python-based processing workspaces (crop, etc.)
├── build_hwpx.py          # Core HWPX assembly engine
├── install.sh             # Dependency installer (macOS/Linux)
├── AGENTS.md              # Repository-wide guidelines and module organization
└── CLAUDE.md              # Detailed project context and developer instructions
```

## Development & Execution

### Prerequisites
- Node.js (>= 20)
- pnpm
- Python 3.10+ (`pip install -r requirements.txt`)
- An AI provider CLI: **Codex CLI (recommended)** or Claude Code CLI — or an API key provider (Claude/OpenAI/DeepSeek SDK, see `.env.example`)

### Key Commands
Run these from the `studio/` directory:

| Command | Description |
| :--- | :--- |
| `pnpm dev` | Starts Next.js dev server on port 3020 |
| `pnpm dev:sse` | Starts the custom SSE server for agent logs |
| `pnpm build` | Production build of the Next.js app |
| `pnpm test` | Runs Vitest unit tests |
| `npx tsc --noEmit` | Validates TypeScript types |
| `pnpm lint` | Runs ESLint |

### Building HWPX (CLI)
You can also run the assembly scripts directly:
```bash
# Fix namespaces in a modified HWPX
python3 build_hwpx.py --input path/to/file.hwpx
```

## Development Conventions

- **Cross-Platform Compatibility:** All code (Node and Python) must support both macOS and Windows.
    - Use `path.join` / `path.resolve` for all file paths.
    - Python detection: `process.platform === "win32" ? "python" : "python3"`.
- **HWPX Manipulation:** HWPX is a ZIP of XML files. After modifying `section0.xml` or other components, you **must** run `fix_namespaces.py` to ensure Hancom Office compatibility.
- **Math Equations:** Follow the strict HWP equation syntax documented in `.claude/skills/hwp-equation/`.
    - Example: `{rmP}_{r} LSUP {n}` for permutations.
- **Testing:** Unit tests live in `__tests__/` folders next to the code. Name them `*.test.ts`.

## Core Pipeline (The "Stages")

1.  **Extractor:** PDF Page -> Image -> Math Problem JSON (Claude).
2.  **Solver/Verifier:** Problem JSON -> Explanation + Answer (Claude, iterative validation).
3.  **Figure Processor:** PDF Crop -> Cleaned Figure (Gemini/nano-banana).
4.  **Builder:** JSON + Figures -> HWPX (Python engine).
5.  **Checker:** HWPX Review -> Feedback Loop (Claude).

For detailed architecture, see `docs/architecture.md` and `AGENTS.md`.

---
> Source: [PNKmath/exam-studio](https://github.com/PNKmath/exam-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
