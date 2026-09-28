## creatorsbrain

> CreatorsBrain is a web application built with Next.js that empowers YouTube creators with AI-powered tools to enhance their video content and workflow. This application leverages various AI models and services to provide features like video analysis, title generation, and thumbnail creation.

# CreatorsBrain: AI-Powered YouTube Assistant

CreatorsBrain is a web application built with Next.js that empowers YouTube creators with AI-powered tools to enhance their video content and workflow. This application leverages various AI models and services to provide features like video analysis, title generation, and thumbnail creation.

## Key Features

- **YouTube Video Analysis:**
  - Fetches the transcript of any YouTube video.
  - Caches transcripts in the database for faster access.

- **AI-Powered Title Generation:**
  - Utilizes OpenAI's GPT-4o-mini model to generate compelling and SEO-friendly video titles.
  - Allows users to provide a summary and specific considerations for title generation.

- **AI-Powered Thumbnail Generation:**
  - Integrates with Replicate and the SDXL model to create custom thumbnails from a text prompt.
  - Stores generated thumbnails in a storage bucket and references them in the database.

- **User Authentication:**
  - Secure user authentication and management powered by Clerk.

- **Backend and Database:**
  - Built on the Convex serverless platform for backend logic and database management.

- **Feature Flagging and Analytics:**
  - Uses Schematic for feature flagging and tracking usage analytics.

## Tech Stack

- **Framework:** [Next.js](https://nextjs.org/) (with Turbopack)
- **UI:** [React](https://react.dev/), [Tailwind CSS](https://tailwindcss.com/), [Radix UI](https://www.radix-ui.com/), [Lucide React](https://lucide.dev/guide/packages/lucide-react) (icons), [Sonner](https://sonner.emilkowal.ski/) (notifications)
- **AI Services:**
  - [OpenAI](https://openai.com/) (GPT-4o-mini) for title generation.
  - [Replicate](https://replicate.com/) (SDXL) for image generation.
  - [@ai-sdk/anthropic](https://www.npmjs.com/package/@ai-sdk/anthropic), [@ai-sdk/react](https://www.npmjs.com/package/@ai-sdk/react), [ai](https://www.npmjs.com/package/ai), [groq-sdk](https://www.npmjs.com/package/groq-sdk) for various AI functionalities.
- **Authentication:** [Clerk](https://clerk.com/)
- **Backend & Database:** [Convex](https://www.convex.dev/)
- **Payments:** [Stripe](https://stripe.com/)
- **YouTube Integration:** [youtubei.js](https://github.com/LuanRT/YouTube.js), [googleapis](https://www.npmjs.com/package/googleapis)
- **Component Library:** [@schematichq/schematic-components](https://www.schematic.com/)
- **Linting & Typing:** [TypeScript](https://www.typescriptlang.org/), [ESLint](https://eslint.org/)

## Getting Started

1. **Clone the repository:**
   ```bash
   git clone https://github.com/your-username/creatorsbrain.git
   cd creatorsbrain
   ```

2. **Install dependencies:**
   ```bash
   npm install
   ```

3. **Set up environment variables:**
   - Create a `.env.local` file by copying the `.env.example` file.
   - Add your API keys and other environment variables for the services used in the application (OpenAI, Replicate, Convex, Clerk, Stripe, etc.).

4. **Run the development server:**
   ```bash
   npm run dev
   ```

5. **Open the application:**
   - Open [http://localhost:3000](http://localhost:3000) in your browser.

## Deployment

This application can be deployed on any platform that supports Next.js. For example, you can deploy it on Vercel, the creators of Next.js.

## Project Structure

- **`/actions`**: Contains server-side actions for handling form submissions and core application logic (e.g., `analyseYoutubeVideo.ts`, `titleGeneration.ts`).
- **`/app`**: The main application directory for Next.js, including pages, layouts, and API routes.
- **`/components`**: Reusable React components used throughout the application.
- **`/convex`**: Configuration and schema for the Convex backend and database.
- **`/features`**: Feature flagging configuration.
- **`/lib`**: Utility functions and libraries used across the application.
- **`/public`**: Static assets like images and fonts.
- **`/types`**: TypeScript type definitions.

---
> Source: [2003nayan/creatorsbrain](https://github.com/2003nayan/creatorsbrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
