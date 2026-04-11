## http-polling-chat

> **Singular Desired Outcome:**

# Project Constitution (gemini.md)

## 0. North Star
**Singular Desired Outcome:**
Build a simple realtime chat web application (Angular + Spring Boot) using HTTP Polling.
- **Goal:** Shared chat room, text messages only, visual appeal, educational focus.
- **Constraints:** HTTP Polling ONLY (No WebSockets), No Auth, Single Room, No Database (In-Memory/H2 implied by "simple" but user said "backend database managed by Spring Boot", assume H2 or simple file for now unless specified, but user said "backend database managed by Spring Boot", usually implies H2 for educational). *Correction*: User said "backend database", will assume generic JPA/H2 for simplicity.

## 1. Data Schemas (The "Payload")

### Core Models

#### Message
```json
{
  "id": 1,
  "sender": "User1",
  "content": "Hello World",
  "timestamp": "2024-01-24T12:00:00Z"
}
```

### API Interface (Contract)

#### 1. Get Messages (Polling Endpoint)
- **GET** `/api/chat/messages`
- **Query Params:** `?sinceId={long}` (Optional, for efficient polling)
- **Response:** `200 OK`
```json
[
  {
    "id": 2,
    "sender": "User2",
    "content": "Hi there",
    "timestamp": "2024-01-24T12:01:00Z"
  }
]
```

#### 2. Send Message
- **POST** `/api/chat/send`
- **Body:**
```json
{
  "sender": "User1",
  "content": "Hello World"
}
```
- **Response:** `200 OK` (Returns the created Message object)

## 2. Behavioral Rules
- **Stack Versioning**: 
    - **Backend**: Spring Boot 2.7.x (Java 8 Compatibility).
    - **Frontend**: Angular (Latest stable compatible with Node 20).
- **Mechanism**: HTTP Polling ONLY. No WebSockets.
- **Auth:** None. Users pick a username (or auto-assigned) and chat.
- **Scope:** Single public chat room. Text only.
- **UI:** Clean, intuitive, visually appealing (Modern CSS/Components).
- **Source of Truth:** Spring Boot Backend (Database).
- **Frontend:** Angular.

## 3. Architectural Invariants
- **3-Layer Architecture**:
    - **Layer 1: Architecture (`architecture/`)**: Technical SOPs.
    - **Layer 2: Navigation**: Logic routing.
    - **Layer 3: Tools (`tools/`)**: Deterministic Scripts.
- **Data-First Rule**: Schema in `gemini.md` is law.
- **Self-Annealing**: Fix errors by updating Architecture rules.

## 4. Maintenance Log
- [2026-01-24] Project Initialized.
- [2026-01-24] Discovery Complete. Schema Defined.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/minh2808)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/minh2808)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
