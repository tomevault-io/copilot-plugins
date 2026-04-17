## cogni-query

> This document outlines the Product Requirements for an interactive document analysis tool. The tool solves the problem of information overload and inefficient knowledge extraction from large or numerous text-heavy documents. It is designed for researchers, legal professionals, business analysts, and students who need to quickly find and synthesize information from dense sources like academic papers, case files, or reports. The value lies in transforming static documents into a dynamic, conversational knowledge base, allowing users to ask complex questions and receive cited, context-aware answers instantly, thereby accelerating research and analysis workflows.

# Overview

This document outlines the Product Requirements for an interactive document analysis tool. The tool solves the problem of information overload and inefficient knowledge extraction from large or numerous text-heavy documents. It is designed for researchers, legal professionals, business analysts, and students who need to quickly find and synthesize information from dense sources like academic papers, case files, or reports. The value lies in transforming static documents into a dynamic, conversational knowledge base, allowing users to ask complex questions and receive cited, context-aware answers instantly, thereby accelerating research and analysis workflows.

# Core Features

- **Batch PDF Upload & Processing**
    - **What it does:** Allows users to upload multiple PDF documents simultaneously through a drag-and-drop interface.
    - **Why it's important:** Users often need to analyze a collection of documents, not just a single file. Batch processing provides critical efficiency.
    - **How it works:** The React frontend sends files to a FastAPI backend endpoint. The backend creates metadata records in MongoDB and dispatches a background processing task for each file to a Celery/Redis queue. This ensures the UI remains responsive regardless of the number or size of the files.
- **AI-Powered Conversational Chat**
    - **What it does:** Provides a chat interface where users can ask natural language questions about the content of the uploaded documents.
    - **Why it's important:** It is the core of the product, offering an intuitive way to query information that is far more powerful than traditional keyword search.
    - **How it works:** This feature is built on a Retrieval-Augmented Generation (RAG) architecture. When a user asks a question, the system embeds the query into a vector, searches a vector database (Qdrant) for the most relevant text chunks from the documents, and then feeds these chunks as context along with the original question to a Large Language Model (LLM).
- **Multi-LLM Support & User-Provided API Keys**
    - **What it does:** Enables users to choose between different LLMs (initially Gemini and OpenAI) and requires them to provide their own API keys for the selected service.
    - **Why it's important:** This offers flexibility to the user and removes the cost and liability of API usage from the service provider. Users can leverage their existing subscriptions and preferred models.
    - **How it works:** The UI will feature a settings panel for users to input and save their API keys. The backend will encrypt these keys before storing them in the database. During a chat session, the backend will decrypt the relevant key for use and instantiate the corresponding LLM client.
- **Verifiable Source Citation**
    - **What it does:** Ensures that every answer provided by the AI is accompanied by a reference to the source document and page number from which the information was derived.
    - **Why it's important:** Citations build trust and allow users to verify the AI's responses, which is critical for academic and professional use cases.
    - **How it works:** During the document processing phase, metadata (source filename, page number) is stored alongside each text chunk. When the RAG system retrieves chunks to form an answer, this metadata is passed along and included in the final response presented to the user.

# User Experience

- **User Personas**
    - **Primary: Alex, a PhD Researcher.** Alex deals with hundreds of academic papers for literature reviews. They need to quickly identify key themes, compare findings across papers, and find specific evidence to support their arguments. They value accuracy and verifiable sources above all else.
    - **Secondary: Sam, a Business Analyst.** Sam reviews long market reports, financial statements, and internal documents to synthesize business insights. They need to quickly extract key figures, identify trends, and summarize lengthy sections. Speed and the ability to query across a "project" of documents are most important.
- **Key User Flows**
    1. **Onboarding & Setup:** A new user lands on the application. They are prompted to navigate to a settings page to enter their OpenAI and/or Gemini API keys. The system validates and saves the keys securely.
    2. **Document Upload & Processing:** The user creates a new project or uses a default workspace. They drag and drop a set of 10 PDF files into the upload area. The UI shows a progress bar for each file, indicating "Uploading," "Processing," and "Completed."
    3. **Conversational Analysis:** Once processing is complete, the user clicks on the chat interface. They type, "Compare the main conclusions of document A and document B regarding topic X." The AI streams back an answer, and below the answer are clickable links labeled "[DocA, pg. 5]" and "[DocB, pg. 12]." Clicking a link highlights the source text.
- **UI/UX Considerations**
    - The interface should be clean and organized, likely a three-panel layout: a collapsible left panel for the document list, a central chat window, and a right panel to display source text.
    - Responses from the AI must be streamed to the user to give immediate feedback and avoid the perception of a "stuck" application.
    - Clear visual cues for the processing status of each document are essential.
    - The user must be able to easily select which documents are included in the current chat context.

# Technical Architecture

- **System Components**
    - **Frontend:** A single-page application built with **React** and **Vite**, using TypeScript for type safety.
    - **Backend API:** A Python-based API built with **FastAPI** to manage file I/O, API logic, and communication between components.
    - **Asynchronous Task Queue:** **Celery** workers using a **Redis** message broker to handle long-running tasks (text extraction, embedding) independently of the API.
    - **Metadata Database:** **MongoDB** to store user data, document metadata (filename, status), and chat histories.
    - **Vector Database:** **Qdrant** to store vector embeddings of document chunks and perform efficient nearest-neighbor searches for the RAG system.
    - **Containerization:** **Docker** and **Docker Compose** will be used to containerize and orchestrate all of the above services for consistent development and deployment environments.
    - **Data Models**
        - **MongoDB `documents` Collection:**`{ _id, user_id, filename, upload_date, status: ('processing' | 'completed' | 'failed') }`
        - **MongoDB `chat_sessions` Collection:**`{ _id, user_id, document_ids: [], start_time, history: [{ role: ('user' | 'ai'), content, sources: [{doc_id, page_num}] }] }`
        - **Qdrant Vector Point Structure:**`Point { id: UUID, vector: [float], payload: { text_chunk: string, document_id: string, page_number: int } }`
    - **APIs and Integrations**
        - **Internal API (FastAPI):**
            - `POST /api/v1/keys`: To submit and update user API keys.
            - `POST /api/v1/documents/upload`: Multipart endpoint for batch PDF uploads.
            - `GET /api/v1/documents`: To list documents and their status.
            - `POST /api/v1/chat`: To handle a new chat message, initiating the RAG workflow. This endpoint will support streaming responses (e.g., using Server-Sent Events).
        - **External Integrations:**
            - OpenAI API (`/v1/chat/completions`)
            - Google Gemini API (`/v1beta/models/gemini-pro:generateContent`)
    - **Infrastructure Requirements**
        - A system capable of running Docker and Docker Compose.
        - The application will be defined entirely within a `docker-compose.yml` file, making the only host dependency Docker itself.

# Development Roadmap

- **Phase 1: Core Engine & Minimal Viability**
    - **Scope:** Focus on the backend processing pipeline and a minimal, non-interactive frontend for testing.
    - **Details:**
        1. Setup `docker-compose.yml` with all services (FastAPI, Mongo, Redis, Qdrant, Celery).
        2. Implement the Celery worker to perform the full PDF processing pipeline: text extraction (`PyMuPDF`), chunking (`LangChain`), embedding generation (`sentence-transformers`), and storage in Qdrant/MongoDB.
        3. Implement the core RAG logic within a FastAPI endpoint. Hardcode a single LLM (e.g., OpenAI) and its API key in the environment for this phase.
        4. Create a simple, non-styled React page that allows uploading a single file and has a basic input field to send a query to the chat endpoint and display the raw JSON response.
        - **Goal:** Prove the end-to-end data flow from PDF upload to a valid, context-based AI response works.
- **Phase 2: Interactive Application (Core Product)**
    - **Scope:** Build the user-facing application and add necessary features for usability.
    - **Details:**
        1. Develop the full React UI: three-panel layout, file upload manager with status tracking, and a functional chat window.
        2. Implement user API key management: a settings page in the UI and secure encryption/storage in the backend.
        3. Add the UI toggle to select between Gemini and OpenAI.
        4. Implement response streaming in the backend and frontend for a smooth chat experience.
        5. Implement source citation display in the UI.
        6. Enable batch uploads and multi-document context selection.
        - **Goal:** A fully functional, interactive application that fulfills all core feature requirements.
- **Phase 3: Enhancements & Polish**
    - **Scope:** Add features that improve the user experience and expand capabilities.
    - **Details:**
        1. Implement full user authentication (e.g., JWT).
        2. Integrate OCR (e.g., Tesseract via a Celery task) to handle scanned PDFs.
        3. Allow users to export chat histories.
        4. Refine text chunking and embedding models for better performance.
        5. Add the ability to delete documents.
        - **Goal:** A robust, polished product ready for a wider audience.

# Logical Dependency Chain

1. **Foundation:** The `docker-compose.yml` configuration is the first and most critical step. All services must be ableto launch and communicate within the Docker network.
2. **Backend Pipeline:** The PDF processing pipeline (Celery worker) must be built next, as no data can be queried without it. This can be developed and tested independently of any UI.
3. **Visible Frontend Shell:** Concurrently, a basic React application shell can be created. The immediate goal is to establish the `FileUpload` component and confirm it can successfully `POST` a file to the FastAPI backend. This provides a visible, testable starting point.
4. **Connect the RAG Loop:** Once files can be uploaded and processed, the chat endpoint logic must be built. The goal is to connect the frontend chat input to the backend, perform the RAG workflow, and get a response back to the UI. The initial version can be unstyled and simply display the raw text response.
5. **Iterate on UX:** With the full loop functional, development should shift to user experience. Implement response streaming, display source citations, add document status notifications, and build out the polished three-panel UI. This atomic approach ensures a working, albeit basic, product exists quickly, which is then iteratively improved.

# Risks and Mitigations

- **Risk: RAG Performance Tuning.** Achieving relevant and accurate answers from the RAG pipeline can be complex and require significant tuning of chunking strategies, embedding models, and prompts.
    - **Mitigation:** Begin with standard, well-documented defaults from libraries like LangChain. Establish a baseline performance, then create a "golden set" of test documents and questions to measure the impact of tuning experiments. Isolate and test each component of the pipeline.
- **Risk: Scope Creep in MVP.** The temptation to add "just one more feature" to the initial build is high and could delay the delivery of a usable core product.
    - **Mitigation:** Adhere strictly to the phased roadmap. Any feature not explicitly listed in Phase 1 is automatically deferred. The singular goal of Phase 1 is to get a single PDF uploaded and queried.
- **Risk: Handling User-Provided API Keys.** Storing and handling user credentials is a significant security responsibility.
    - **Mitigation:** Keys will never be stored in plain text. Use a strong, non-reversible encryption library (e.g., `cryptography.fernet`) with a secret key managed outside of version control (`.env` file or production secrets manager). Keys should only be decrypted in memory at the moment of use.

# Appendix

- **Core Python Libraries:** `FastAPI`, `Celery`, `PyMuPDF`, `langchain`, `sentence-transformers`, `qdrant-client`, `cryptography`, `pymongo`.
- **Core Frontend Libraries:** `React`, `Vite`, `Zustand`, `axios`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mo7amedElfadil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
