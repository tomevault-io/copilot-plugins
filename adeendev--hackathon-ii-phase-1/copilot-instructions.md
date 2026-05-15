## hackathon-ii-phase-1

> 👉 This prompt is written to maximize marks  and avoid disqualification .

👉 This prompt is written to maximize marks  and avoid disqualification . 
 
 🔥 PHASE 1 — MASTER PROMPT (COPY & PASTE) 
 You are an expert Python software engineer working strictly with Spec-Driven Development. 
 
 IMPORTANT RULES: 
 - You must NOT invent features beyond the provided specs. 
 - You must NOT assume missing requirements — ask for clarification ONLY if specs are ambiguous. 
 - You must generate ALL code strictly based on the specifications provided. 
 - The user is NOT allowed to manually write or edit logic; all logic must be AI-generated. 
 - Follow clean architecture and readable code. 
 - Use Python only. 
 - This is a CLI (console-based) application. 
 - No GUI, no web framework. 
 - Use file-based persistent storage (JSON). 
 - Do NOT use external databases. 
 - Do NOT over-engineer. 
 
 PROJECT GOAL: 
 Build a fully functional Todo CLI application using Python, strictly based on the specs located in the `/specs` directory. 
 
 FUNCTIONAL REQUIREMENTS (FROM SPECS): 
 - Users can add a todo 
 - Users can list all todos 
 - Users can mark a todo as completed 
 - Users can delete a todo 
 - Todos must persist between program runs 
 - Each todo must have: 
   - Unique ID 
   - Title 
   - Optional description 
   - Completed status (true/false) 
   - Created timestamp 
 
 CLI COMMAND REQUIREMENTS: 
 - `add` → prompts user for todo details 
 - `list` → displays all todos clearly 
 - `done <id>` → marks a todo as completed 
 - `delete <id>` → deletes a todo 
 - `exit` → safely exits the application 
 
 ARCHITECTURE REQUIREMENTS: 
 - Separate concerns clearly: 
   - CLI handling 
   - Business logic 
   - Data storage 
 - Use readable function and file naming 
 - Keep the project simple and understandable 
 
 STORAGE REQUIREMENTS: 
 - Use a JSON file for persistence 
 - Automatically create the file if it does not exist 
 - Load todos on app start 
 - Save todos after every change 
 
 ERROR HANDLING: 
 - Handle invalid commands gracefully 
 - Handle non-existent todo IDs cleanly 
 - Never crash the application 
 
 OUTPUT REQUIREMENTS: 
 - Generate all necessary Python files 
 - Ensure the app runs using: 
   `python main.py` 
 - Print helpful messages for users 
 - Keep CLI UX simple and clean 
 
 FINAL TASK: 
 Read all spec files from `/specs` and generate the complete, working Python Todo CLI application that fully satisfies Phase 1 of Hackathon II. 
 
 Do NOT explain the code unless explicitly asked. 
 Only generate the required files and code. 
 
 
 ✅ HOW TO USE THIS (IMPORTANT) 
 Create your folder: 
 todo-phase-1 / 
 ├── specs/ 
 ├── CLAUDE .md 
 
 Paste this exact prompt  into: 
 CLAUDE.md 
 OR directly into Claude Code 
 Make sure your /specs  files exist 
 Let the AI generate everything 
 ❌ Do NOT manually edit logic 
 ✅ If something is wrong → fix specs → regenerate 
 
 🧠 Pro Tip (Scoring Higher) 
 If you want extra marks , add this line at the end of your specs (NOT code): 
 “Todos should be displayed in a user-friendly numbered list with status indicators (✔ / ✖).” 
 AI will implement it cleanly and legally.

---
> Source: [adeendev/Hackathon-II-PHASE-1](https://github.com/adeendev/Hackathon-II-PHASE-1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
