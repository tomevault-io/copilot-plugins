## prompt-engineering-project

> For building the entire project using AI

# ApexGift – Full Project Requirements (React + Node.js Version)
For building the entire project using AI

## 1. PROJECT DESCRIPTION

ApexGift is a simple ChatGPT-like web application where:
- The user can chat with the AI
- The user can upload a file (.txt, .docx, .pdf)
- The AI extracts personality, interests, and emotions
- The AI generates personalized gift ideas + message
- Sessions are stored in history
- UI is very simple and clean, like ChatGPT

The entire project must be built using AI tools (Gemini, ChatGPT, Claude).

## 2. WHAT WE MUST BUILD

### A. FRONTEND (React)
React + Vite or React + CRA or React + Next → You choose, Gemini will set it up

**Core Components**
- **Sidebar**
    - Chat history list
    - “New Chat” button
    - Theme switch (dark/light)
    - Settings button
- **Chat UI**
    - Scrollable message list
    - User bubble (left)
    - AI bubble (right)
- **Input Section**
    - Text input
    - File upload button
    - Send button
- **Result Sections (AI Responses)**
    - Extracted context
    - Gift recommendations
    - Personalized message

### B. BACKEND
You can use Node.js + Express (recommended for simplicity).

**Gemini should generate:**
- **API Endpoints**
    - `POST /api/extract`
        - Input: text
        - Output: JSON with interests, personality, tone
        - Uses Extraction Prompt
    - `POST /api/gift`
        - Input: extracted JSON
        - Output: 3–5 gift ideas
    - `POST /api/message`
        - Input: extracted JSON
        - Output: personalized message
    - `POST /api/upload`
        - Input: file
        - Output: extracted text

**Backend will call:**
- Gemini API
- or OpenAI API
- or any LLM endpoint

### C. DATABASE
Choose one:
- **Option 1: Supabase (easiest)**
    - Used for:
        - Storing chat sessions
        - Storing chat messages
        - Storing context extraction result
- **Option 2: Firebase Firestore**
    - Same usage as above.

## 3. PROJECT FEATURES

### A. Chat System
- Real-time chat-like UX
- Press Enter → send
- File upload triggers automatic extraction

### B. Chat History
- Stored in DB:
    - `chat_id`
    - `timestamp`
    - list of messages
    - AI pipeline results
    - file content (optional)
- Sidebar shows:
    - Each chat title (first AI message summary)
    - Click → loads chat

### C. File Upload
- Support: `.txt`, `.pdf`, `.docx`
- Steps:
    1. User uploads file
    2. Client sends to backend
    3. Backend extracts text
    4. Sends extraction → gift → message prompts
    5. Returns final result to frontend

## 4. PROMPT PIPELINE (MUST BE IMPLEMENTED)

### A. Extraction Prompt
```text
You are an AI that analyzes text to detect personality, interests, emotional tone, relationship type, and important details.

Extract:
- Key interests
- Personality traits
- Emotional tone
- Communication style
- Important events or memories
- Relationship type
Return JSON only.
```

### B. Gift Recommendation Prompt
```text
Based on this JSON profile, generate 3–5 personalized gift ideas.
Each idea must include a short explanation.
```

### C. Message Generator Prompt
```text
Write a short personalized message matching the user's tone.
Make it friendly, natural, and thoughtful.
```

## 5. TECH STACK SUMMARY
- **Frontend**: React, TailwindCSS (or ChakraUI or MaterialUI), Axios (for API calls)
- **Backend**: Node.js + Express, Gemini API or OpenAI API, pdf-parse, mammoth for file extraction
- **Database**: Supabase or Firebase

## 6. GEMINI GENERATION TASKS

### 1. Project Structure
Generate a full folder tree:
- `/frontend`
    - `/src`
        - `/components`
        - `/pages`
        - `/hooks`
        - `/utils`
- `/backend`
    - `/routes`
    - `/controllers`
    - `/services`
    - `/prompts`
- `/database`
- `/docs`

### 2. FRONTEND
Generate full working React code:
- Sidebar component
- Chat UI
- MessageBubble component
- FileUpload component
- ChatHistory component
- Theme toggle
- API service (Axios)
- Full App.jsx

### 3. BACKEND
Generate full Node.js Express code:
- `server.js`
- routes for:
    - `POST /api/extract`
    - `POST /api/gift`
    - `POST /api/message`
    - `POST /api/upload`
- Controllers + services
- File parsing logic
- API key middleware
- All endpoints should use the 3-step AI pipeline: extract, recommend, message

### 4. DATABASE
Choose Supabase or Firebase and generate:
- tables
- schema
- CRUD functions for: `createChat`, `saveMessage`, `listChats`

### 5. PROMPTS
Include the exact prompts for:
- Extraction
- Gift recommendation
- Message generation

### 6. Running Instructions
Generate instructions for:
- Installing frontend
- Installing backend
- Connecting database
- Running the system

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jubams)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jubams)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
