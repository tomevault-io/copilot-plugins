## trinity-wealth-engine

> > **Project:** `invest-agents` (Trinity-Wealth-Engine)

# CLAUDE.md — กฎเหล็กและคู่มือสถาปัตยกรรมระบบ (Trinity Wealth Engine)

> **Project:** `invest-agents` (Trinity-Wealth-Engine)
> **Scope:** เอกสารฉบับนี้เป็น "กฎเหล็กและแนวทางปฏิบัติเชิงวิศวกรรม" สำหรับ Claude Code และ AI Coding Agent ในการพัฒนา ดูแลรักษา และปรับปรุงระบบ
> **Core Principle:** ยึดมั่นในความถูกต้องทางการเงิน (Financial Integrity), สถาปัตยกรรม Hexagonal Architecture, และเสถียรภาพระดับ Production (High Resilience & Distributed Safety)

---

## 1. ⚙️ สภาพแวดล้อมและคำสั่งหลัก (Environment & CLI Commands)

* **ระบบปฏิบัติการ:** Windows
* **เทคโนโลยีหลัก:** Python 3.11+, FastAPI, React 19, Vite, TypeScript, SQLite, LangGraph/LangChain
* **การจัดการ Dependency:** `uv` (Fast Python Package Manager) และ `npm`

### 🚀 คำสั่งจัดการระบบ
* **ติดตั้ง / ซิงค์ Python Dependencies:**
  ```powershell
  uv sync
  ```
* **รัน Backend Server (FastAPI + Outbox Workers):**
  ```powershell
  .venv\Scripts\python -m uvicorn api.main:app --port 8000 --reload
  ```
* **รัน Frontend Dev Server (React + Vite):**
  ```powershell
  npm --prefix web run dev
  ```
* **รัน Automated Test Suites:**
  * Backend Pytest (สถาปัตยกรรม, Unit, Integration):
    ```powershell
    .venv\Scripts\pytest tests/ -v
    ```
  * Frontend Vitest:
    ```powershell
    npm --prefix web test -- --run
    ```
  * Frontend TypeScript Check:
    ```powershell
    npm --prefix web run typecheck
    ```
* **สร้าง / ปรับปรุง Golden Baseline Manifests:**
  ```powershell
  .venv\Scripts\python scripts/generate_golden_manifests.py
  ```

---

## 2. 🧠 Mandatory Plan-First Workflow (กฎเหล็ก: วางแผนก่อนลงมือทำ)

1. **ห้ามแก้ไขไฟล์ทันที:** เมื่อได้รับโจทย์ที่ซับซ้อน (สร้างฟีเจอร์, ปรับสถาปัตยกรรม, แก้ไขบั๊กข้ามชั้น) **ห้าม** แตะต้องไฟล์โค้ดทันที
2. **เสนอแผนงานก่อนเสมอ (Execution Plan):** ร่างแผนการทำงานทีละขั้นตอน (Step-by-Step) เป็นภาษาไทย โดยระบุ:
   - 📂 ไฟล์ที่จะแก้ไขหรือสร้างใหม่ (ระบุ Path ชัดเจน)
   - ⚙️ Logic หรือ Interface ที่จะเปลี่ยนแปลง
   - ⚠️ ผลกระทบต่อ Layer อื่น และความเสี่ยง (Risks & Mitigations)
   - 🧪 แผนการทดสอบเพื่อยืนยันความถูกต้อง (Verification Plan)
3. **รอการอนุมัติ (Wait for Approval):** หยุดรอจนกว่าผู้ใช้จะอนุมัติ ("Approve", "OK", "เห็นด้วย", "ลุยเลย")
4. **ลงมือทำแบบศัลยกรรม (Surgical Changes):** แก้ไขเฉพาะจุดที่จำเป็น เคารพสไตล์เดิม ไม่ refactor โค้ดรอบข้างโดยไม่จำเป็น และไม่ทิ้ง Dead Code

---

## 3. 🏛️ สถาปัตยกรรม Hexagonal Architecture & 16 AST Rules

ระบบถูกตรวจสอบความถูกต้องเชิงสถาปัตยกรรมอัตโนมัติผ่าน 16 AST Rules ใน `tests/architecture/test_dependency_rules.py` โดยมีโครงสร้างแบ่งแยกชั้นดังนี้:

```text
[ Inbound Routers (api/routers/) ]    [ Background Workers (api/workers/) ]
                  │                                     │
                  ▼                                     ▼
         [ Application Services / Ports / DTOs (application/) ]
                                  │
                                  ▼
                    [ Domain Entities / Core Logic (core/) ]
                                  ▲
                                  │
[ Driven Adapters: Obsidian Vault | SQLite DB | LLMs | Data APIs (tools/ | api/db/) ]
```

### 3.1 กฎการแยก Layer (Layer Isolation Rules)
1. **Domain Layer (`core/`, `domain/`):** เป็นศูนย์กลางของ Business Rules ห้าม Import ชั้นนอก (Infrastructure, Application, API, หรือ Adapters) เด็ดขาด
2. **Application Layer (`application/`):** จัดการ Use Case Workflows, Ports (Interfaces), DTOs และ Saga Orchestrators **ห้าม Import Database Direct Query, SQLite Connection หรือ HTTP Frameworks ตรงๆ**
3. **Driven Adapters (`tools/`, `infrastructure/`):** ทำหน้าที่ Implement Ports ที่ Application กำหนด (เช่น `ObsidianEarningsCallAdapter`, `SqliteEarningsCallWorkflowAdapter`)
4. **Inbound Adapters (`api/routers/`):** ทำหน้าที่รับ HTTP Request, Validate ข้อมูล และส่งต่อให้ Application Service **ห้ามทำ Filesystem I/O ใน Router เด็ดขาด**
5. **DAO Pattern (`api/db/repositories/`):** ห้าม DAO เรียก `.commit()` หรือ `.rollback()` เองภายในฟังก์ชัน การเปิด-ปิด Transaction ต้องถูกควบคุมในระดับ Service หรือ Unit of Work
6. **Background Workers (`api/workers/`):** ต้องทำงานผ่าน Application Services และ Outbox Repositories เท่านั้น ห้ามเรียก State DB โดยตรง

---

## 4. 🔄 Distributed Safety: Saga & Transactional Outbox Pattern

เมื่อมีกระบวนการที่ต้องทำงานข้ามระบบที่ไม่สามารถ Commit ใน Transaction เดียวกันได้ (เช่น Obsidian Markdown Vault + LLM API + SQLite Database / Kanban):

### 4.1 Saga State Machine & Transactional Outbox
* **Saga States:** `NEW` $\rightarrow$ `SUMMARIZED` $\rightarrow$ `NOTE_WRITTEN` $\rightarrow$ `KANBAN_PENDING` $\rightarrow$ `COMPLETED` / `FAILED`
* **Transactional Outbox:** ทุกครั้งที่เขียน Obsidian Note สำเร็จ ต้องบันทึกสถานะและสร้าง Outbox Event ภายใน Atomic DB Transaction เดียวกัน เพื่อรับประกันว่างานจะไม่สูญหาย (At-Least-Once Delivery)
* **Outbox Worker:** Background Worker ดึง Event ไปส่งมอบ พร้อมระบบเช่าเวลา (Lease), กู้คืนงานค้าง (Lease Expiry Recovery), และนับจำนวน Retry สูงสุด

### 4.2 Idempotency & Fencing Token Lease
* **Deterministic Source Key:** คำนวณ Idempotency Key จาก canonical ticker, canonical period, transcript hash, และ prompt version เพื่อป้องกันการรันซ้ำ
* **Execution Lease Fencing:** เมื่อมี Request ซ้ำเข้ามาพร้อมกัน เฉพาะผู้ที่ถือ `owns_execution = True` (Fencing Token) เท่านั้นที่มีสิทธิ์เรียก LLM และเขียน Obsidian ส่วน Caller อื่นจะได้รับสถานะเดิมกลับไป (ป้องกัน Duplicate LLM Calls และ Race Conditions)

### 4.3 HTTP Status Code Contract
* **`200 OK`:** กระบวนการทำงานเสร็จสิ้นสมบูรณ์ (Workflow Completed)
* **`202 Accepted`:** รับคำขอเรียบร้อย อยู่ระหว่างประมวลผลเบื้องหลัง (In Progress / Kanban Pending) พร้อมส่ง Run State ให้ Client ทำ Polling
* **`422 Unprocessable Content`:** ข้อมูล Request ไม่ผ่านการตรวจสอบของ Domain/Schema Validation
* **`503 Service Unavailable`:** ผู้ให้บริการภายนอก (LLM Provider / External API) ไม่พร้อมใช้งานหรือ Timeout
* **`404 Not Found`:** ไม่พบ Resource หรือ Entity (พร้อมการตรวจสอบ Ticker Ownership Isolation)

### 4.4 OpenAPI Golden Manifest Rule
* **ห้ามแก้ไขไฟล์ `tests/fixtures/manifest_openapi_schema.json` ด้วยมือเด็ดขาด**
* ทุกครั้งที่มีการเพิ่ม/แก้ไข API Router หรือ Schema ให้รัน `.venv\Scripts\python scripts/generate_golden_manifests.py` เพื่อสร้าง Baseline Manifest ใหม่หลังตรวจสอบความถูกต้องแล้ว

---

## 5. 🛡️ ความถูกต้องทางการเงินและข้อมูล (Data Integrity Invariants)

กฎเหล่านี้คือ **Invariants ที่ห้ามละเมิดเด็ดขาด**:

### 5.1 Atomic Storage Mutations (การบันทึกไฟล์ปลอดภัย)
* ทุกฟังก์ชันที่เขียนไฟล์ลง Obsidian Vault หรือไฟล์การเงิน ต้องเขียนลง **ไฟล์ชั่วคราว (Shadow/Temp file)** ก่อนเสมอ
* ใช้ **OS-level Atomic Swap** (`os.replace()` หรือ `_atomic_write_text()`) ในการสลับไฟล์จริง เพื่อป้องกันไฟล์เสียหายหากไฟดับหรือระบบ Crash กลางคัน

### 5.2 Anti-Drift Bottom-Up Recalculation Loop
* ทุกครั้งที่มีการเปลี่ยนแปลงข้อมูลรายการสินทรัพย์ (Asset Mutation):
  1. **Per-Asset Layer:** คำนวณ Market Value, Cost Basis, Unrealized P/L รายตัว
  2. **Summary Layer:** Re-sum ยอดรวมพอร์ตทั้งหมดจากระดับสินทรัพย์ (ห้าม Patch เฉพาะส่วนต่าง Delta)
  3. **Allocation Layer:** Re-calculate สัดส่วน Asset Allocation % ใหม่ทั้งหมด
  4. **Commit:** จึงบันทึกผลลง Storage แบบ Atomic

### 5.3 No In-Context Financial Math (ห้าม LLM คำนวณเลขในใจ)
* **ห้ามให้ LLM คำนวณตัวเลขทางการเงินใน Prompt หรือ Context เด็ดขาด** (เช่น บวก ลบ กำไรขาดทุน คำนวณดอกเบี้ย สัดส่วนพอร์ต)
* ตัวเลขทุกตัวต้องถูกคำนวณผ่าน **Deterministic Python Tools** เท่านั้น LLM มีหน้าที่เลือกว่าจะเรียก Tool ใดด้วย Argument อะไร

### 5.4 Single Source of Truth & Clean Cache
* แหล่งความจริงเดียวของข้อมูลพอร์ตคือไฟล์ใน Obsidian Vault (`memories/`)
* SQLite ทำหน้าที่เป็น Index และ Cache เสริมความเร็ว โดยต้องมีกลไก Invalidation หรือ Rebuild จาก Vault เสมอ

### 5.5 PII Gateway & Secret Safety
* ข้อมูล PII ต้องถูกแปลง/ปกปิดผ่าน `core/security.py` ก่อนส่งไปยัง External LLM เสมอ ห้าม Bypass
* ห้าม Hardcode API Keys ลงในโค้ด ต้องโหลดผ่าน `.env` และ `os.getenv()` เสมอ

---

## 6. 🤖 มาตรฐาน Multi-Agent, Tools & Observability

### 6.1 Agent Roles & Boundaries
* **Supervisor / Manager:** ควบคุม Routing และ State Transition ตามผลลัพธ์ของ Tools ไม่คำนวณตัวเลขเอง
* **Bookkeeper:** เอนทิตีเดียวที่ดูแล Structured State (พอร์ตการลงทุน, Ledger, Holdings)
* **Archivist:** ดูแล Unstructured Content (สรุปบทวิเคราะห์, ข่าว, Earnings Calls, YouTube Transcripts)
* **Macro Quant & Strategic Allocator:** คำนวณ Macro Metrics, Guardrails, และ Portfolio Asset Stance

### 6.2 Tool Interface & Self-Correction Loop
* **Native Tool Calling:** กำหนดคำอธิบาย วิธีใช้งาน และ Argument Constraints ใน **Google Style Docstrings** ของ `@tool` เพื่อให้โมเดลผูก Schema อัตโนมัติ (ห้ามยัดลง System Prompt)
* **Tool Error Handling:** ฟังก์ชัน `@tool` (Interface Layer) **ห้ามปล่อย Exception ทะลุกลับหา LLM** ให้ Catch และ Return เป็น Error String (เช่น `"Error: ..."` หรือ `validation_error(...)`) เพื่อให้ LLM เกิด Self-Correction Loop ในการแก้ไข Argument
* **Centralized Model Registry:** ห้าม Hardcode ชื่อโมเดล ให้ดึงผ่าน `core.model_registry` (เช่น `get_model_name("extractor")`)

### 6.3 Communication & Logging Standards
* **Prefix Token Structure:** ข้อความ Log และ Agent Communication ต้องกวาดสายตาเข้าใจง่าย (Scannable) และ Parse สะดวก คั่นด้วย ` | `:
  ```text
  [ACTION] | [KEY_DELTA_OR_METRIC] | [CURRENT_CONTEXT_STATUS]
  ```
  *ตัวอย่าง:* `[BUY AAPL] | qty +10 @ 225.50 | cash $15,400 -> $13,145`

---

## 7. 🧹 สุขอนามัยของ Workspace (Workspace Cleanliness)

1. **ห้ามสร้างไฟล์ขยะบน Root Directory:** ไฟล์เทสต์ ชั่วคราว หรือ Artifacts ต้องเก็บใน `tests/` หรือ `scratch/` เท่านั้น
2. **Whitespace & Formatting Integrity:** ตรวจสอบให้ `git diff --check` ผ่านเสมอ ห้ามมี Trailing Whitespace หรือข้อผิดพลาดเรื่อง Line Ending
3. **Green Test Gate:** ก่อนส่งมอบงานทุกครั้ง โค้ดต้องผ่านการทดสอบครบถ้วน:
   - Architecture AST Rules: `pytest tests/architecture/test_dependency_rules.py`
   - OpenAPI Contract: `pytest tests/api/test_openapi_contract.py`
   - Frontend Typecheck & Tests: `npm --prefix web run typecheck` และ `npm --prefix web test`

---

> *"Code is read far more often than it is written. Optimize for correctness, resilience, and architectural integrity."*

---
> Source: [gnoMarkII/Trinity-Wealth-Engine](https://github.com/gnoMarkII/Trinity-Wealth-Engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
