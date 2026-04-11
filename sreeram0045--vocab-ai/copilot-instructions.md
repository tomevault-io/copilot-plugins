## vocab-ai

> VocabAI 2.0 is an interactive educational application that helps users learn complex vocabulary through the lens of popular TV shows. It uses generative AI to define words, create fictional scenarios involving characters from suitable TV universes (e.g., "The Big Bang Theory" for science words, "Succession" for corporate drama), and generate visual representations of those scenarios.

# VocabAI 2.0

## Project Overview

VocabAI 2.0 is an interactive educational application that helps users learn complex vocabulary through the lens of popular TV shows. It uses generative AI to define words, create fictional scenarios involving characters from suitable TV universes (e.g., "The Big Bang Theory" for science words, "Succession" for corporate drama), and generate visual representations of those scenarios.

### Tech Stack

*   **Framework:** Next.js 16.1.1 (App Router)
*   **Language:** TypeScript
*   **Styling:** Tailwind CSS v4, Shadcn/UI components (Radix UI + Lucide React)
*   **AI Integration:**
    *   **Text & Logic:** `meta-llama/Llama-3.1-8B-Instruct` (via Hugging Face Inference API)
    *   **Image Generation:** `black-forest-labs/FLUX.1-schnell` (via Hugging Face Inference API)
*   **SDKs:** Vercel AI SDK (`ai`, `@ai-sdk/openai`)

## Building and Running

### Prerequisites

*   Node.js (v20+ recommended)
*   Hugging Face API Token

### Environment Setup

Create a `.env.local` file in the root directory and add your Hugging Face token:

```env
HF_TOKEN=your_hugging_face_token_here
# or
HUGGINGFACE_API_KEY=your_hugging_face_token_here
```

### Scripts

*   **Development Server:**
    ```bash
    npm run dev
    ```
    Runs the app at `http://localhost:3000`.

*   **Build for Production:**
    ```bash
    npm run build
    ```

*   **Start Production Server:**
    ```bash
    npm run start
    ```

*   **Linting:**
    ```bash
    npm run lint
    ```

## Key Files & Structure

*   **`app/page.tsx`**: The main application interface. It handles user input, manages the loading states for text and image generation, and displays the results (definition, synonyms, conversation, and generated image).
*   **`app/api/generate/route.ts`**: The backend logic for text generation.
    *   Accepts a word.
    *   Uses Llama 3.1 to determine the word's meaning, select a thematic TV universe, write a short script, and create a visual prompt.
    *   Validates the output using `zod` schema validation.
*   **`app/api/image/route.ts`**: The backend logic for image generation.
    *   Accepts a visual prompt and a universe name.
    *   Constructs a style-specific prompt (e.g., "in the visual style of Friends").
    *   Calls the FLUX.1 model on Hugging Face to generate the image.
    *   Returns the image as a Base64 string.
*   **`components/ui/`**: Contains reusable UI components (Buttons, Cards, Inputs, etc.) built with Radix UI and Tailwind CSS.

## Development Conventions

*   **State Management:** Local React state (`useState`) is used in `app/page.tsx` for simplicity.
*   **Styling:** Utility-first CSS with Tailwind. Components generally follow the Shadcn/UI patterns.
*   **Error Handling:** Both frontend and backend implement error handling. The backend returns JSON error messages which are displayed in the UI via an Alert component.
*   **Type Safety:** TypeScript interfaces (like `VocabData`) and Zod schemas are used to ensure data consistency between the AI output and the frontend.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sreeram0045)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sreeram0045)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
