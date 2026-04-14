## memutrader

> This project is a sample application demonstrating the integration of **MemU** (an agentic memory layer for LLM applications). It consists of a React-based frontend and a Flask-based backend that acts as an interface to the MemU API.

# Project: Mem U Trader

## Overview
This project is a sample application demonstrating the integration of **MemU** (an agentic memory layer for LLM applications). It consists of a React-based frontend and a Flask-based backend that acts as an interface to the MemU API.

## Tech Stack

### Frontend
- **Framework**: React 19 (via Vite)
- **Language**: TypeScript
- **Styling**: Tailwind CSS, PostCSS
- **UI Components**: Radix UI (via shadcn/ui patterns), Lucide React
- **Build Tool**: Vite

### Backend
- **Framework**: Flask (Python)
- **Dependencies**: flask-cors, requests, openai
- **External Service**: MemU API (v3), OpenAI API

## Directory Structure

```
memu_sample/
├── frontend/           # React application
│   ├── src/
│   │   ├── components/ # UI components (Button, Input, etc.)
│   │   ├── lib/        # Utilities
│   │   ├── App.tsx     # Main application component
│   │   └── main.tsx    # Entry point
│   ├── package.json
│   └── vite.config.ts
├── backend/            # Flask server
│   └── app.py          # API routes (/memories, /memorize)
└── sample_scripts/     # Standalone Python scripts for testing MemU API
```

## Setup & Running

### Prerequisites
- Node.js (v18+ recommended)
- Python 3.x

### 1. Backend Server
The backend runs on port **5001**.

1. Navigate to the backend directory:
   ```bash
   cd backend
   ```

2. Set up environment variables:
   ```bash
   cp .env.example .env
   ```
   Edit `.env` and set your `API_KEY` (MemU) and `OPENAI_API_KEY`.

3. Install dependencies and run the server:
   ```bash
   pip install -r requirements.txt
   python app.py
   ```

### 2. Frontend Application
The frontend development server typically runs on port **5173**.

```bash
cd frontend
npm install
npm run dev
```

## API Interface Details

The Flask backend acts as a proxy to the MemU API, handling authentication and request formatting.

### 1. Retrieve Memories
- **Endpoint**: `GET /memories`
- **Description**: Retrieves relevant memories from MemU based on a query, then streams an AI-generated financial analysis (BUY/SELL/HOLD decision) based on those memories.
- **Request Parameters** (Query String):
  - `user_id` (string, required): The ID of the user.
  - `agent_id` (string, optional): The ID of the agent context.
    - *Note*: If not provided, the backend attempts to infer the relevant currency pair (e.g., "USD/JPY") from the `query` using OpenAI and formats it as `agent_USD/JPY`.
  - `query` (string, required): Search query text to find relevant memories and ask for advice.
- **Response Format**: Server-Sent Events (SSE)
  - **Event: `memories`**: JSON object containing retrieved memories from MemU API.
  - **Event: `token`**: Streamed text tokens from OpenAI providing the analysis (Action and Reason).
- **Example Usage**:
  ```bash
  curl -N "http://localhost:5001/memories?user_id=demo_user&query=Should%20I%20buy%20USD/JPY?"
  ```

### 2. Memorize Interaction
- **Endpoint**: `POST /memorize`
- **Description**: Stores a user interaction into MemU memory.
- **Request Body** (JSON):
  - `user_id` (string, required): The ID of the user.
    - *Note*: Backend automatically prepends `user_` prefix before sending to MemU.
  - `agent_id` (string, required): The ID of the agent context (e.g., currency pair).
    - *Note*: Backend automatically prepends `agent_` prefix before sending to MemU.
  - `user_message` (string, required): The content text to be memorized.
- **Response**: JSON object confirming the memory task submission from MemU API.
- **Example Usage**:
  ```bash
  curl -X POST http://localhost:5001/memorize \
       -H "Content-Type: application/json" \
       -d '{
         "user_id": "demo_user",
         "agent_id": "USD/JPY",
         "user_message": "The resistance level is 150.00"
       }'
  ```

## Environment Configuration
The application requires the following environment variables in `backend/.env`:
- `API_KEY`: Your MemU API Key (v3).
- `OPENAI_API_KEY`: Your OpenAI API Key.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/theMistletoe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
