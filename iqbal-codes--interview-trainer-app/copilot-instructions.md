## interview-trainer-app

> * **Framework:** Next.js (React)

# **AI-Powered Interview Practice Platform \- Project Rules for Cursor AI (Multi-Provider: OpenAI & Gemini)**

## **1\. Core Technology Stack**

* **Framework:** Next.js (React)  
* **Language:** TypeScript (strict mode preferred)  
* **Styling:** Tailwind CSS  
* **UI Components:** shadcn@latest  
* **State Management:** Zustand (or a similar lightweight React state manager)  
* **Backend/API:** Next.js API Routes, WebSocket Server (e.g., using ws package) for real-time communication.  
* **Database & BaaS:** Supabase (PostgreSQL, Supabase Auth, Supabase Storage)  
* **Voice AI Pipeline:** Custom implementation supporting **both** OpenAI and Google Gemini.  
  * **STT/LLM/TTS (Real-time):**  
    * **OpenAI:** OpenAI Realtime API (via openai SDK).
    * **Google Gemini:** Google Gemini Live API (via @google/generative-ai SDK).  
  * **LLM (Non-streaming):** Vercel AI SDK (@ai-sdk/google) is used for non-streaming tasks like initial question generation or batched feedback analysis.  
* **Real-time Communication:** WebSockets.

## **2\. Code Generation & Style**

* **Modularity:** Generate code in a modular fashion. Create reusable React components and well-defined service functions (e.g., in lib/).  
* **TypeScript:** All new code should be in TypeScript. Use appropriate types for props, state, and function arguments. Avoid any.  
* **Clarity & Comments:** Generated code should be clear. Add JSDoc comments for complex functions, API routes, and service logic.  
* **Error Handling:** Implement robust try/catch blocks for all API calls, database operations, and WebSocket communications.  
* **Async/Await:** Use async/await for asynchronous operations.  
* **Naming Conventions:** Follow standard JavaScript/TypeScript naming conventions.

## **3\. Frontend (Next.js & shadcn@latest)**

* **Component Structure:** Follow standard Next.js page and component structure.  
* **Real-time Interaction:**  
  * Use the browser's MediaRecorder API to capture microphone audio.  
  * Use the native WebSocket API to establish and manage a connection with the backend server.  
  * Implement logic to stream audio chunks to the server.  
  * Implement logic to receive and handle incoming streams from the server (e.g., live transcript, TTS audio chunks).  
  * Create a client-side audio player to queue and play incoming TTS audio buffers with minimal latency.

## **4\. Backend (Next.js API Routes & WebSocket Server)**

* **REST APIs:** Standard API Routes will handle non-real-time operations like user auth and session generation (/api/interviews/generate).  
* **WebSocket Server:**  
  * Manage the lifecycle of WebSocket connections.  
  * Receive incoming audio streams from clients.  
  * Orchestrate the real-time STT-LLM-TTS pipeline by routing requests to the appropriate AI provider service.  
* **Authentication:** Secure both REST API routes and WebSocket connections (e.g., via an initial auth handshake over WebSockets).  
* **Validation:** Validate all incoming data from both REST and WebSocket clients.

## **5\. Supabase Interaction**

* **Client SDK:** Use the official Supabase JavaScript client library (@supabase/supabase-js) for all database, auth, and storage interactions.  
* **RLS:** Assume Row Level Security policies are in place. Write queries with the understanding that users can only access their own data.

## **6\. Voice AI Pipeline Integration (OpenAI & Google Gemini)**

* **Provider Abstraction:** All real-time voice AI interactions on the backend must go through an abstraction layer (e.g., a RealtimeVoiceProvider interface). We will create separate service classes (GeminiRealtimeService, OpenAIRealtimeService) that implement this interface.  
* **Provider Selection:** The active AI provider will be determined by an environment variable (e.g., ACTIVE\_AI\_PROVIDER='GOOGLE'). The WebSocket server will instantiate the correct service class based on this variable.  
* **Streaming:** Leverage the native streaming capabilities of the chosen provider's SDK for both LLM responses and TTS audio generation.  
* **Interruption Handling (Barge-in):** The architecture must support interruption. The backend handler for the active provider must be able to receive an "interrupt" signal and stop the ongoing LLM/TTS process.  
* **Environment Variables:** All API keys (OPENAI\_API\_KEY, GOOGLE\_API\_KEY, etc.) and secrets must be stored securely as environment variables.

## **7\. General Guidelines**

* **Refer to Provided Documents:** When working on a feature, refer to the relevant PRD, Feature Spec, Data Model, API Spec, or UI Wireframe that I provide context for. Our core architectural plan is now the realtime\_ai\_guide\_v2 document.  
* **Iterative Development:** We are following an adjusted, phased approach. Focus on the specific task for the current phase, starting with the Gemini implementation.  
* **Ask for Clarification:** If a request is ambiguous, ask for clarification.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iqbal-codes) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
