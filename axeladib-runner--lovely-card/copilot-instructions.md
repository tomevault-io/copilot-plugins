## lovely-card

> https://<your-domain>/api/v1


# Interactive Birthday Card – REST API (v1)

**Base URL**
```
https://<your-domain>/api/v1
```

**Media type**
```
Content-Type: application/json
Accept: application/json
```

**Authentication**
- **Recipient session**: create a session using an invitation token (see `POST /sessions`). Server sets an **HttpOnly cookie** `session` (scope: `recipient`, card-bound).
- **Admin**: send **Firebase ID token** in `Authorization: Bearer <idToken>`.

**Errors (uniform format)**
```json
{ "error": { "code": "STRING_CODE", "message": "Readable message", "details": {} } }
```

**Common status codes**
200 OK · 201 Created · 204 No Content · 400 Validation · 401 Unauthorized ·
403 Forbidden · 404 Not Found · 409 Conflict · 429 Too Many Requests · 500 Server Error

---

## Resources & Endpoints

### 1) Sessions
**Create Session**
```
POST /sessions
```

### 2) Cards
- `GET /cards/{cardId}` – Get card
- `POST /cards` – Create card (admin)
- `PATCH /cards/{cardId}` – Update card (admin)
- `DELETE /cards/{cardId}` – Delete card (admin)

### 3) Slides
- `GET /cards/{cardId}/slides` – List slides
- `POST /cards/{cardId}/slides` – Create slide (admin)
- `PATCH /cards/{cardId}/slides/{slideId}` – Update slide (admin)
- `DELETE /cards/{cardId}/slides/{slideId}` – Delete slide (admin)

### 4) Questions
- `GET /cards/{cardId}/questions` – List questions
- `POST /cards/{cardId}/questions` – Create question (admin)
- `PATCH /cards/{cardId}/questions/{questionId}` – Update question (admin)
- `DELETE /cards/{cardId}/questions/{questionId}` – Delete question (admin)

### 5) Answers
- `POST /cards/{cardId}/answers` – Submit answer

### 6) Prizes
- `GET /cards/{cardId}/prizes` – List prizes
- `POST /cards/{cardId}/prize-selections` – Pick a prize
- `POST /cards/{cardId}/prizes` – Create prize (admin)
- `PATCH /cards/{cardId}/prizes/{prizeId}` – Update prize (admin)
- `DELETE /cards/{cardId}/prizes/{prizeId}` – Delete prize (admin)

### 7) Events
- `POST /cards/{cardId}/events` – Log progress/event

### 8) Invitations
- `POST /invitations` – Create invitation
- `GET /invitations/{linkId}` – Get invitation
- `DELETE /invitations/{linkId}` – Delete invitation

### 9) Media (Uploads)
- `POST /media/uploads` – Admin signed upload

### 10) Analytics
- `GET /cards/{cardId}/analytics` – Get analytics

---

## Example Flows

**Recipient playback**
1. POST /sessions
2. GET /cards/{cardId}
3. GET /cards/{cardId}/slides
4. GET /cards/{cardId}/questions
5. POST /cards/{cardId}/answers
6. POST /cards/{cardId}/prize-selections
7. POST /cards/{cardId}/events

**Admin authoring**
1. POST /cards
2. POST /cards/{cardId}/slides
3. POST /cards/{cardId}/questions
4. POST /cards/{cardId}/prizes
5. POST /invitations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/axeladib-runner) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
