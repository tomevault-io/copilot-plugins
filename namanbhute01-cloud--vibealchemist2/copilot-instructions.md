## vibealchemist2

> 1. **Environment & Network Standards**

# Project: Vibe Alchemist V2 Integration Rules

1. **Environment & Network Standards**
   - **Rule**: Enforce 127.0.0.1 over localhost for all CORS and Redirect URI configurations.
   - **Backend**: Update CORSMiddleware in FastAPI to allow `http://127.0.0.1:5173` and `http://127.0.0.1:8080`.
   - **Frontend**: Set `VITE_API_URL` to `http://127.0.0.1:8080`.

1. **Environment & Network Standards**
   - **Rule**: Enforce 127.0.0.1 over localhost for all CORS and Redirect URI configurations.
   - **Backend**: Update CORSMiddleware in FastAPI to allow `http://127.0.0.1:5173` and `http://127.0.0.1:8080`.
   - **Frontend**: Set `VITE_API_URL` to `http://127.0.0.1:8080`.

# 4. **WebSocket Implementation**
   - **Action**: Create a `/ws/vibe-stream` endpoint in FastAPI.
   - **Payload**: Send a JSON object: `{ "emotion": string, "suggested_vibe": float }`.

5. **Error Handling**
   - **Action**: If Google Drive credentials are missing (as seen in logs), suppress the warning and default to MemoryCache instead of attempting to write to a non-existent cloud path.

---
> Source: [namanbhute01-cloud/VibeAlchemist2](https://github.com/namanbhute01-cloud/VibeAlchemist2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
