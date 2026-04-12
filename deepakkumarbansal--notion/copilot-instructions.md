## notion

> Project: Backend system for a Notion clone with integrated AI knowledge architect features.

📌 1. Overview
Project: Backend system for a Notion clone with integrated AI knowledge architect features.
Goal: To design and implement a scalable, robust, and performant backend using the MERN stack (MongoDB, Express.js, Node.js) that supports core Notion-inspired functionalities and specified AI enhancements, with a phased approach to achieve long-term feature parity and pixel-perfect frontend support.
Guiding Principles:
MVP First: Prioritize a functional core for initial phases.
Scalability: Design for growth.
Performance: Optimize for low latency.
Modularity: Structure for maintainability.
Security: Implement robust protection.
Data Integrity: Ensure consistency.
🔧 2. Core Backend Architecture (MERN)
2.1. Node.js Runtime: Latest stable LTS version.
2.2. Express.js Framework:
Middleware: Morgan (logging), Helmet (security headers), CORS, express.json() (body parsing), compression.
Routing: Modular route handlers.
2.3. MongoDB Database:
Mongoose ODM: For schema definition, validation, and business logic.
2.4. Authentication & Authorization:
JWT for stateless sessions.
Password hashing (bcrypt).
RBAC (Owner, Admin, Editor, Viewer) - basic roles for MVP, expanded later.
2.5. Real-time Communication:
Strictly Post-MVP: WebSockets (e.g., Socket.IO) for real-time collaboration. No WebSocket implementation in initial phases.
2.6. API Design:
RESTful principles, JSON format.
Versioning: /api/v1/....
API Documentation (Swagger/OpenAPI planned for post-MVP).
2.7. General Note for Coding Agent: The backend must aim to provide data structures that comprehensively support all attributes visible or used by the Notion frontend for its features. This means schemas may need to be expanded iteratively. For MVP, focus on data for core functionality.
📄 3. Data Models (MongoDB Schemas using Mongoose - MVP Focus)
3.1. User Collection:
_id: ObjectId
name: String
email: String (unique, indexed)
password: String (hashed)
avatarUrl: String (optional)
oauthProviders (Post-MVP): [{ providerName: String, providerId: String }]
createdAt: Date
updatedAt: Date
3.2. Workspace Collection:
_id: ObjectId
name: String
ownerId: ObjectId (ref: 'User', indexed)
icon: String (emoji or URL, optional)
members (MVP): [{ userId: ObjectId (ref: 'User'), role: String (enum: ['owner', 'editor']) }]
rootPageIds: [ObjectId (ref: 'Page')]
createdAt: Date
updatedAt: Date
3.3. Page Collection:
_id: ObjectId
workspaceId: ObjectId (ref: 'Workspace', indexed)
authorId: ObjectId (ref: 'User')
title: String (text-indexed)
icon: String (optional)
coverImage: String (optional, URL)
contentOrder: [ObjectId (ref: 'Block')] (Blocks stored in separate Block collection)
parentPageId: ObjectId (ref: 'Page', nullable, indexed)
childPageOrder: [ObjectId (ref: 'Page')]
tags (MVP - manual): [String] (text-indexed)
createdAt: Date
updatedAt: Date (indexed)
Permissions, locking, version history are Post-MVP.
3.4. Block Collection (MVP Block Types):
_id: ObjectId
pageId: ObjectId (ref: 'Page', indexed)
workspaceId: ObjectId (ref: 'Workspace', indexed)
type: String (enum: ['paragraph', 'heading1', 'heading2', 'heading3', 'bulleted_list_item', 'numbered_list_item', 'todo_list_item', 'code_block', 'quote', 'image', 'divider', 'link_to_page'], indexed)
content: Mixed (structure defined per type below)
paragraph: { text: String }
heading1/heading2/heading3: { text: String }
bulleted_list_item/numbered_list_item: { text: String }
todo_list_item: { text: String, checked: Boolean (default: false) }
code_block: { code: String, language: String (optional) }
quote: { text: String }
image: { url: String, caption: String (optional) }
divider: {} (empty object)
link_to_page: { linkedPageId: ObjectId (ref: 'Page') }
properties: Mixed (MVP: basic styling like color, background_color if simple; otherwise Post-MVP)
childrenOrder: [ObjectId (ref: 'Block')] (for nested list items, toggles - toggles are Post-MVP)
createdAt: Date
updatedAt: Date
Synced blocks are Post-MVP.
3.5. Database Collection (Post-MVP Phase):
Initial MVP will not include Notion-style databases. This schema is for future reference.
_id: ObjectId
blockId: ObjectId (ref: 'Block', unique, indexed)
workspaceId: ObjectId (ref: 'Workspace', indexed)
title: String
propertySchema (Post-MVP): [{ id: String, name: String, type: String (e.g., 'text', 'number', 'select'), options (Post-MVP): { choices: [{ id: String, name: String, color: String }] for 'select' } }]
viewDefinitions (Post-MVP): [{ id: String, name: String, type: String ('table'), config (Post-MVP): { visiblePropertyIds: [String], sort: [{ propertyId: String, direction: 'asc'|'desc' }] } }]
3.6. DatabaseItem Collection (Post-MVP Phase):
Corresponds to Database collection.
3.7. Embedding Collection (for AI features - initial phase):
_id: ObjectId
workspaceId: ObjectId (ref: 'Workspace', indexed)
resourceId: ObjectId (indexed) (Page.id for MVP)
resourceType: String (enum: ['page'], indexed)
chunkText: String (text-indexed)
vector: [Number]
modelUsed: String
createdAt: Date
Note: For MVP, embeddings stored in MongoDB. Basic cosine similarity or Atlas Vector Search if available. Dedicated Vector DB is Post-MVP.
⚙️ 4. API Endpoints (MVP Focus)
4.1. Authentication (/api/v1/auth)
POST /register
POST /login (email/password only for MVP)
POST /logout
GET /me
4.2. Workspaces (/api/v1/workspaces)
POST /: Create workspace.
GET /: List user's workspaces.
GET /:workspaceId: Get workspace details.
PUT /:workspaceId: Update workspace (name, icon).
DELETE /:workspaceId (soft delete preferred for MVP).
POST /:workspaceId/members: Add member (MVP: editor role).
DELETE /:workspaceId/members/:userId: Remove member.
4.3. Pages (/api/v1/workspaces/:workspaceId/pages)
POST /: Create page (title, parentPageId optional).
GET /:pageId: Get page details (including contentOrder of block IDs).
PUT /:pageId: Update page (title, icon, coverImage, parentPageId, contentOrder array of block IDs).
DELETE /:pageId (soft delete preferred for MVP).
4.4. Blocks (/api/v1/blocks)
POST /: Create block (requires pageId, type, content).
GET /:blockId: Get block details.
PUT /:blockId: Update block content and properties.
DELETE /:blockId.
POST /:blockId/move: (MVP) Reorder block within its current direct parent's childrenOrder or Page.contentOrder. Changing parent is Post-MVP.
4.5. Search (/api/v1/workspaces/:workspaceId/search)
GET /?q=<query>: (MVP) Basic text search on Page.title, Page.tags, and Block.content.text (for text-based blocks).
4.6. AI Services (/api/v1/ai) - Initial AI Features
POST /:workspaceId/pages/:pageId/generate-tags: (MVP) Trigger auto-tag generation. Uses OpenAI API (e.g., GPT-3.5-turbo). Tags are suggestions, user confirms.
POST /:workspaceId/qna: (MVP) Workspace Q&A. Uses OpenAI embeddings (e.g., text-embedding-ada-002) and GPT-3.5-turbo for RAG. Context chunks from MongoDB Embedding collection.
Content Chunking (MVP): Pages chunked by paragraphs or N words (e.g., 200) with overlap (e.g., 20).
Prompt (MVP): "Based only on the following context, answer the question. If the answer isn't in the context, state that. Context: {CONTEXT_CHUNKS}. Question: {USER_QUESTION}. Answer:"
Context Handling (MVP): Retrieve top K=3 chunks.
AI Auto-Linker & Knowledge Graph are Post-MVP.
🧠 5. AI Feature Backend Requirements (MVP Focus)
5.1. Auto Tag Generator (MVP):
API endpoint as in 4.6.
Integration with OpenAI API (GPT-3.5-turbo with specific prompting).
Service to temporarily store suggestions; frontend handles user confirmation to update Page.tags.
5.2. Workspace Q&A System (MVP):
API endpoint as in 4.6.
Background process (or on-demand for MVP) to chunk page content, generate embeddings (OpenAI text-embedding-ada-002), and store in Embedding collection.
RAG pipeline: query embedding, vector search in MongoDB (cosine similarity or Atlas Vector Search), LLM API call (GPT-3.5-turbo) with dynamic prompt.
🔒 6. Security Requirements (MVP)
Secure password storage (bcrypt).
Protection against common web vulnerabilities (XSS, CSRF - Helmet helps). Mongoose helps with NoSQLi.
Basic input validation for API requests.
HTTPS for all communication.
Proper CORS configuration.
Secure storage of OpenAI API key.
⚙️ 7. Performance & Scalability Requirements (Initial Considerations)
Database Optimization: Proper indexing on MongoDB collections for common query patterns (as noted in schemas).
Asynchronous Operations (MVP - limited): Embedding generation for Q&A might be an initial candidate for an async job if page saves become slow. Otherwise, most operations are synchronous for MVP.
Stateless Services: Design API services to be stateless.
🛠️ 8. Deployment & Operations (Initial Setup)
Containerization (Docker) recommended.
CI/CD pipeline (e.g., GitHub Actions) for automated builds and deployments.
Centralized logging (e.g., platform-provided like Vercel/Heroku logs, or simple console logging for local dev).
Environment configuration management.
🌊 9. Phased Implementation Plan (Backend - Vibe-Driven Sequence)
Phase Alpha: Core Foundation
Node.js/Express.js project structure.
MongoDB/Mongoose setup.
Schemas: User, Workspace, basic Page/Block.
Auth APIs (register, login, JWT).
Basic CRUD APIs: Workspaces, Pages (metadata only).
Phase Beta: Content & Structure
Full Page/Block schemas for MVP block types.
APIs for CRUD of blocks within pages, contentOrder updates.
Page hierarchy APIs (parent/child linking).
Basic Search API.
Phase Gamma: Initial AI Integration
Backend for Auto Tag Generator (OpenAI integration, suggestion flow).
Backend for Workspace Q&A (embedding pipeline for pages, RAG logic, OpenAI integration).
Phase Delta: Polish & Stability
Address bugs, improve API stability based on usage.
Basic performance tuning.
Prepare for next wave of features (e.g., more block types, early database considerations).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deepakkumarbansal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/deepakkumarbansal)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
