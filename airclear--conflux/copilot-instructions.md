## conflux

> A React-based web application that facilitates a multi-model "discussion" between two AI agents (**Cognito** and **Muse**) to solve user queries. The application features a shared **Notepad** that AI models can collaboratively edit to refine the final output.

# Dual AI Chat - Project Overview

A React-based web application that facilitates a multi-model "discussion" between two AI agents (**Cognito** and **Muse**) to solve user queries. The application features a shared **Notepad** that AI models can collaboratively edit to refine the final output.

## Technical Stack
- **Framework:** React 19 (TypeScript)
- **Build Tool:** Vite
- **AI Integration:** 
  - Google Gemini SDK (`@google/genai`)
  - OpenAI-compatible API support via custom fetch implementation
- **Styling:** Tailwind CSS
- **Utilities:**
  - `marked` & `dompurify`: Markdown rendering and sanitization
  - `katex`: Mathematical expression rendering
  - `lucide-react`: Iconography
  - `diff`: Content comparison

## Core Concepts & Architecture

### Discussion Flow
The application implements a "Research -> Discussion -> Synthesis" workflow:
1. **User Query:** The user submits a prompt (and optionally an image).
2. **Initial Cognito Response:** The "Logic" model provides an initial breakdown.
3. **Discussion Loop:** Cognito and Muse exchange messages (either for a fixed number of turns or until they agree to stop) to refine the solution.
4. **Final Answer:** Cognito synthesizes the discussion into a comprehensive final answer, which is then used to update the shared Notepad.

### Key Roles
- **Cognito (Logical AI):** Focuses on factual accuracy, logical breakdown, and synthesis.
- **Muse (Creative AI):** Focuses on creative expansion, critiquing Cognito, and providing alternative perspectives.
- **Notepad:** A shared state that acts as the "final product" workspace. AI models can perform specific actions (replace, append, search & replace) on the notepad via a structured JSON protocol embedded in their responses.

### Key Directories
- `components/`: UI components (ChatPanel, Notepad, Settings, etc.)
- `hooks/`: Specialized logic hooks:
  - `useAppController.ts`: Central orchestrator.
  - `useChatLogic.ts`: Manages the discussion loop and AI turn-taking.
  - `useNotepadLogic.ts`: Handles notepad state and AI-driven modifications.
  - `useSettings.ts`: Manages API keys, model selection, and discussion parameters.
- `services/`: API client wrappers for Gemini and OpenAI.
- `utils/`: 
  - `promptBuilder.ts`: Logic for constructing complex multi-turn prompts.
  - `aiResponseParser.ts`: Extracts spoken text and notepad instructions from AI responses.

## Building and Running

### Prerequisites
- Node.js (Latest LTS recommended)
- A Gemini API Key (or an OpenAI-compatible API key)

### Commands
- **Install Dependencies:** `npm install`
- **Start Development Server:** `npm run dev` (Runs on `localhost:5173`)
- **Build for Production:** `npm run build`
- **Preview Production Build:** `npm run preview`

### Configuration
Environment variables can be set in a `.env.local` file:
- `API_KEY`: Your Google Gemini API key.

Alternatively, users can configure API keys and custom endpoints directly in the application's "Settings" modal.

## Development Conventions

### AI Interaction Protocol
When adding features that involve AI, ensure:
1. **Prompt Consistency:** Use `utils/promptBuilder.ts` to maintain consistent system instructions across different models.
2. **JSON Handling:** AI models are instructed to return a JSON block for notepad updates. Ensure any new prompt includes the `NOTEPAD_INSTRUCTION_PROMPT_PART` from `constants.ts`.
3. **Type Safety:** Update `types.ts` when modifying the communication protocol between models or the notepad action schema.

### UI/UX
- The application uses a split-screen layout (`ResizeHandle.tsx`) to balance Chat and Notepad visibility.
- **Mobile Support:** Uses a tabbed interface (`activeMobileTab`) to switch between Chat and Notepad on small screens.
- **Thinking Process:** Supports Gemini's internal "thinking" process, which is rendered in a specialized UI block within the chat.

## TODO / Future Enhancements
- [ ] Add support for more granular notepad operations.
- [ ] Implement local history persistence for notepad versions.
- [ ] Expand model support for other providers (e.g., Anthropic).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/airclear)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/airclear)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
