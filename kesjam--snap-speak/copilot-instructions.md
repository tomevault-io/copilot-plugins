## snap-speak

> This file provides **Cursor AI** with guidelines and context for generating code and configuring the **Voice-Driven Photo & Document Manager**. It outlines the project scope, folder structure, dependencies, and specific prompts to ensure Cursor’s AI functions optimally for this project.

# .cursorrule.md

## Purpose
This file provides **Cursor AI** with guidelines and context for generating code and configuring the **Voice-Driven Photo & Document Manager**. It outlines the project scope, folder structure, dependencies, and specific prompts to ensure Cursor’s AI functions optimally for this project.

---

## Project Overview

- **Project Name**: Voice-Driven Photo & Document Manager  
- **Goal**: 
  1. Create an iOS app (Swift/SwiftUI) for **voice-powered** photo capture and metadata tagging.  
  2. Develop a React-based **web dashboard** to view, search, and manage uploaded photos and documents.  
  3. Use **Firebase** for data storage, authentication (optional), and file storage.  
- **Primary Features**:
  - Voice Command Parsing (iOS):  
    - “Take photo of Floor 2, Unit 205” → tags photo with `floor=2, unit=205`.  
    - “Show me photos of Floor 1” → retrieves all photos with `floor=1`.  
  - Photo Storage:  
    - Upload photos to **Firebase Storage**, store metadata in **Firestore**.  
  - Web Dashboard:  
    - Display a list or gallery of tagged photos.  
    - Basic search functionality (by floor, unit, or custom tags).
  - Optional Document Linking:  
    - Attach PDFs or drawings to specific floors/units.

---

## Guidelines for Cursor AI

1. **Language & Framework Prompts**  
   - **Swift/SwiftUI** for iOS:
     - Use Apple’s Speech Framework for voice recognition.  
     - Integrate Firebase SDK (Core, Firestore, Storage).  
   - **React** for Web:
     - Use the `create-react-app` or `Vite` template.  
     - Integrate Firebase (Firestore & Storage) using `firebase` npm package.

2. **Code Generation Approach**  
   - Provide **concise** and **commented** code snippets.  
   - Whenever user prompts for code, generate **self-contained** functions or components that are easy to copy into the project.  
   - **Avoid** extraneous logic or advanced patterns beyond the user’s scope (the user is a beginner).

3. **Directory Structure**  
   - **iOS** (Swift) Project:
     ```
     VoicePhotoManager-iOS/
     ├─ VoicePhotoManager.xcodeproj
     ├─ VoicePhotoManager/
     │   ├─ App.swift
     │   ├─ ContentView.swift
     │   └─ FirebaseConfig.swift (for Firebase init)
     └─ Podfile
     ```
   - **Web** (React) Project:
     ```
     VoicePhotoManager-Web/
     ├─ src/
     │   ├─ App.js
     │   ├─ firebase.js (Firebase init)
     │   ├─ components/
     │   └─ pages/
     ├─ package.json
     └─ .gitignore
     ```

4. **Required Dependencies**  
   - iOS (in `Podfile`):
     ```ruby
     pod 'Firebase/Core'
     pod 'Firebase/Firestore'
     pod 'Firebase/Storage'
     pod 'Firebase/Auth' # if authentication is needed
     ```
   - Web (`package.json`):
     ```json
     {
       "dependencies": {
         "firebase": "^<latest_version>",
         "react": "^<latest_version>",
         "react-dom": "^<latest_version>"
       }
     }
     ```

5. **Sample Prompts for Cursor**  
   - **Voice Recognition Setup (Swift)**  
     ```
     // Prompt: "Generate Swift code using Apple's Speech framework 
     // to transcribe short voice commands. Return the recognized text."
     ```
   - **Firebase Upload Function (Swift)**  
     ```
     // Prompt: "Generate Swift code to upload an image to Firebase Storage 
     // and save its metadata (floor, unit) in Firestore."
     ```
   - **React Photo Gallery**  
     ```
     // Prompt: "Generate a React component named 'PhotoGallery' that fetches
     // photo metadata from Firestore, displays images, and shows floor/unit tags."
     ```
   - **Voice Search (React)**  
     ```
     // Prompt: "Generate a React hook that uses the Web Speech API 
     // to capture voice input, convert to text, and filter a photo list accordingly."
     ```

6. **Implementation Details**  
   - **Transcription Parsing**:  
     - Simple approach: parse recognized text for keywords like “Floor” and “Unit.”  
     - Example: "Floor 2, Unit 205" → `floor=2`, `unit=205`.
   - **CRUD Operations**:
     - iOS and React should be able to:
       - **Create** (upload photos with metadata).  
       - **Read** (display items).  
       - **Update** (change tags if needed).  
       - **Delete** (remove photos or documents).
   - **Testing & Debugging**:  
     - Use the iOS simulator for basic tests (voice recognition might require a physical device).  
     - For React, run locally with `npm start` or `yarn start`.

7. **Styling & UI**  
   - Keep the UI **simple** and **intuitive** (the user is a beginner).  
   - SwiftUI for iOS handles layout nicely; for React, basic CSS or a library like Tailwind can be used if desired.

8. **Deployment**  
   - **iOS**: Deploy using TestFlight or direct device debugging.  
   - **Web**: Deploy on **Firebase Hosting** or any static hosting provider.

---

## Additional Notes

- **Keep Code Simple**: The user is new to programming, so avoid overly abstract or advanced patterns.  
- **Always Provide Comments**: Each generated snippet should include short comments explaining key lines of code.  
- **Focus on Core Requirements**: Priority is voice capture → tagging → uploading → searching. Anything else can be added later.  

When receiving user prompts in Cursor, ensure to:

Output minimal, well-organized code.
Include inline comments for clarity.
Provide short usage instructions if needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kesjam) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
