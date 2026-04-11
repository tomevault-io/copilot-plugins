## echofy

> I want you to build a modern, professional web application that runs a local Whisper transcription workflow in the cloud. Here’s exactly what I want:

I want you to build a modern, professional web application that runs a local Whisper transcription workflow in the cloud. Here’s exactly what I want:

1. **Purpose**: The app allows users to upload audio files (.m4a, .mp3, .wav) and automatically transcribe them into editable text using OpenAI’s Whisper model. The transcription should be editable in the browser before saving.

2. **Backend Requirements**:
   - Use Python (Flask or FastAPI) for server-side.
   - Use OpenAI’s Whisper library to transcribe audio.
   - Load the Whisper model once on startup to improve speed.
   - Detect device type (CPU/GPU) and use the best available device (`mps` on Mac, GPU if available, CPU fallback otherwise).
   - Accept file uploads via POST.
   - Return transcription as editable HTML text (rich-text or textarea).

3. **Frontend Requirements**:
   - Use modern, responsive UI framework (Tailwind CSS recommended) to create a **clean, Nexus-style interface**.
   - File upload card should be visually prominent with drag-and-drop optional.
   - When transcription starts, show a **loading overlay** with:
       - Spinner animation
       - Progress bar (simulated if real-time progress isn’t available)
       - “Transcribing… Please wait” message
   - Once transcription finishes:
       - Display the text in a **rich-text editable area** (like Quill or TinyMCE)
       - Include buttons: Save as `.txt`, Copy to clipboard
   - Optionally show metadata: audio duration, language detected

4. **Workflow**:
   - User uploads audio
   - Spinner + progress overlay appears
   - Whisper transcribes the audio
   - Text is returned to the frontend in an editable box
   - User can edit, then save/export
   - Ensure long audio files are handled gracefully (simulate progress, maybe chunk in backend if needed)

5. **UX/UI Details**:
   - Glassmorphism or smooth card design
   - Gradient backgrounds
   - Smooth transitions and responsive layout
   - Optional dark/light mode toggle
   - Works on desktop and mobile

6. **Extra Features to Consider**:
   - Model selector: small / medium / large
   - Auto-detect language
   - Timestamps per line or sentence
   - Session-based history of uploads and transcripts
   - Download transcript as `.txt` or `.docx`

7. **Constraints**:
   - All processing should remain local to the server instance (audio files are not sent to any external service)
   - Must be fully functional with minimal dependencies beyond Flask/FastAPI, Whisper, Tailwind, and lightweight JS libraries.

8. **Deliverables**:
   - Complete backend code (Python + Flask/FastAPI)
   - Frontend HTML/CSS/JS integrated
   - Instructions for deploying to cloud environment
   - Clear comments explaining how audio flows from upload → Whisper → editable text → save/export

Output the **full working code** in one response and explain any deployment considerations for cloud hosting. Make the UI look professional, modern, and interactive as described.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cDilash)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/cDilash)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
