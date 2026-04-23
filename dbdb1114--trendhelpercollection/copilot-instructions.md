## trendhelpercollection

> > 이 문서는 Claude가 이 저장소에서 작업할 때 따라야 하는 **계약(Contract)** 과 **작업 절차(Playbook)** 를 정의합니다.

# CLAUDE.md — Root (Contracts & Playbook)

> **Purpose**  
> 이 문서는 Claude가 이 저장소에서 작업할 때 따라야 하는 **계약(Contract)** 과 **작업 절차(Playbook)** 를 정의합니다.  
> 처음 파이썬/데이터 파이프라인을 만드는 개발자도 이해할 수 있도록 **설명 + 명령 + 완료기준(DoD)** 를 명확히 적습니다.

---

## Versioning Policy
- **Current Target:** **v1 (MVP: 최소 수직 슬라이스)**
- v1 범위를 벗어나는 제안/의견은 **`TODO:V2`** 로만 표기하고 지금은 구현하지 않는다.
- 질문은 **최대 1회**까지만 하고, 합리적 가정을 세워 **즉시 진행**한다.

### v1 — 최소 수직 슬라이스
1) **수집:** KR 인기급상승 50건 **1회 수집** → `videos` upsert + `video_metrics_snapshot` 스냅샷 insert  
2) **분석:** **Velocity**(Δviews/Δminutes) 계산 → 상위 N 추출  
3) **생성:** Claude로 **제목 3~5개 + 태그 5~10개** 생성(스키마 검증, 가드레일 적용)  
4) **서비스:** FastAPI **`/health`**, **`/ideas`** 두 엔드포인트만 우선 공개

### v2 — 이후 확장 (설계만 표시)
- APScheduler 주기 수집/증분, Google Trends/댓글  
- NLP(토큰화/불용어/표기 통합) → TF-IDF(+SVD) → KMeans 주제 버킷  
- `/trending`, `/ideas/batch`, 인증/레이트리밋, 메트릭/알림, Redis 캐시

---

## Ground Rules (항상 지켜야 함)
- **타임존:** 모든 시간은 **UTC**, DB 컬럼은 `TIMESTAMPTZ`.
- **시크릿:** API 키/DB URL은 **`.env`** 로부터만 로드. 하드코딩 금지.
- **로깅:** **JSON 라인** 포맷(UTC). 에러/핵심 로그에 `trace_id` 포함.
- **형식 검증:** API 응답/생성 결과는 **Pydantic 스키마**로 엄격 검증. 실패 시 자동 재시도.
- **파일 경계:**  
  - `migrations/versions/*` 기존 파일 **수정 금지**(새 리비전 생성)  
  - **전체 파일 재작성 금지**, **patch/diff** 형태로 변경 제안
- **허용 라이브러리(예시):** `fastapi`, `uvicorn`, `sqlalchemy`, `psycopg`, `alembic`, `httpx`, `pandas`, `scikit-learn`, `konlpy`, `apscheduler`, `pydantic`, `pydantic-settings`, `redis`  
  (새 라이브러리는 지금은 **`TODO:V2`**)

---

## Repository Map (요약)
- `collection/` 수집, `analysis/` 분석, `generation/` 생성, `app/` API, `core/` 공용(DB/로깅), `migrations/` Alembic
- 각 모듈 폴더에 **별도 `CLAUDE.md`** 가 있으며, 이 **루트 규칙을 상속**한다.

---

## Contracts (v1)

### 1) Database (PostgreSQL)
- **Table: `videos`**
  - `video_id TEXT PK`
  - `title TEXT`, `description TEXT`, `channel TEXT`, `category TEXT`, `tags JSONB`, `country_code TEXT`
  - `published_at TIMESTAMPTZ`
- **Table: `video_metrics_snapshot`**
  - `id BIGSERIAL PK`
  - `video_id TEXT FK → videos.video_id`
  - `captured_at TIMESTAMPTZ`  *(수집 시각, UTC)*
  - `view_count BIGINT`, `like_count BIGINT`, `comment_count BIGINT`
  - **Index:** `(video_id, captured_at)`  *(정렬·중복 방지 용도)*

> v2에서 `topic_buckets`, `generated_ideas` 등을 추가할 수 있음(지금은 설계만).

---

### 2) Analysis — Velocity (v1)
- **Input:** `video_metrics_snapshot(video_id, captured_at, view_count)`
- **Computation:**  
  - 같은 `video_id` 그룹 정렬 후  
    `views_per_min = diff(view_count) / diff(captured_at_minutes)`  
  - `diff(captured_at) <= 0` 행 **drop**  
  - 음수/무한대 제거, 상위 1% **클리핑**(이상치 컷)
- **Output:** 상위 N `(video_id, views_per_min)` JSON (또는 임시 테이블)
- **DoD:** 상위 10 JSON을 콘솔 출력 혹은 파일에 저장 가능

---

### 3) API — `/ideas` (v1)
- **Request JSON**
```json
{
  "video_id": "string or null",
  "keywords": ["아이폰17","루머"],
  "signals": {"views_per_min": 123.4},
  "style": {"tone":"info","language":"ko","length_sec":20}
}
```
- **Response JSON** *(Pydantic로 검증)*  
```json
{
  "titles": ["...","...","..."],
  "tags": ["#아이폰17","#루머","#테크"],
  "script_beats": {"hook":"...","body":"...","cta":"..."},
  "metadata": {"model":"claude-3-...","safety_flags":[]}
}
```
- **DoD:** 200 OK & 스키마 검증 통과, 금칙어/길이/이모지 제한 위반 시 **자동 재생성**

---

### 4) Generation Guardrails (v1)
- **제목:** 20–35자, 과도한 낚시성/과장 금지, 숫자 남용 금지, 이모지 ≤ 1
- **태그:** 핵심 3 + 롱테일 2~7, 금칙어/PII 금지
- **스크립트:** 3-Beat (Hook → Body → CTA), **사실 기반·간결**

---

## Setup & Commands (macOS, v1)

### 0) 의존성(최소)
```bash
python -m venv .venv && source .venv/bin/activate
pip install -U pip
pip install fastapi "uvicorn[standard]" sqlalchemy "psycopg[binary]" alembic httpx pandas scikit-learn konlpy apscheduler pydantic pydantic-settings redis
```

### 1) DB 마이그레이션 (초기 스키마)
> **Claude에게 요청:** `core/models`에 위 두 테이블 모델 작성 →  
> `alembic revision --autogenerate` → `alembic upgrade head` 를 **patch/diff** 로 제안

수동 실행:
```bash
alembic revision --autogenerate -m "init v1 tables"
alembic upgrade head
```

### 2) 수집(1회) — KR 트렌딩 50
> **Claude에게 요청:**  
> `collection/clients/youtube.py`에 YouTube 클라이언트,  
> `collection/jobs/collector_trending.py` 작성(차트=mostPopular, region=KR, limit=50).  
> `videos` upsert + `video_metrics_snapshot` insert. **재실행 idempotent**.

실행:
```bash
python collection/jobs/collector_trending.py --country KR --limit 50
```

### 3) 분석 — Velocity
> **Claude에게 요청:**  
> `analysis/jobs/analyzer_velocity.py` 작성. groupby-sort-diff, 이상치 클리핑, 상위 10 JSON 출력.

실행:
```bash
python analysis/jobs/analyzer_velocity.py --window 3
```

### 4) 생성 — 제목/태그 최소
> **Claude에게 요청:**  
> `generation/jobs/generate_ideas.py` 작성. 위 **/ideas** 스키마로 제목 3~5, 태그 5~10 생성.  
> 길이/금칙어/이모지 **가드레일** + Pydantic 검증 + 실패 재시도.

실행:
```bash
python generation/jobs/generate_ideas.py --video-id <some_video_id>
```

### 5) API — 최소 공개
> **Claude에게 요청:**  
> `app/api/ideas.py` 라우터 추가, `/ideas` 구현 → `app/main.py`에 include. `/health` 유지.

실행:
```bash
uvicorn app.main:app --reload
curl http://localhost:8000/health
```

---

## Review Checklist (v1 DoD)
- [ ] `alembic upgrade head` 성공(스키마 반영)  
- [ ] 수집 1회로 `videos` 50행 + `video_metrics_snapshot` 스냅샷 생성  
- [ ] velocity 상위 10 JSON 출력(Δt=0/음수/∞ 처리됨)  
- [ ] `/ideas`가 JSON 스키마대로 응답, 위반 시 자동 재생성 동작  
- [ ] 로그가 **JSON 라인**으로 남고 `trace_id` 포함

---

## Logging (JSON 라인 예시)
```python
# core/logging.py
import json, logging, sys, time

class JsonFormatter(logging.Formatter):
    def format(self, record):
        base = {
            "ts": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
            "level": record.levelname,
            "logger": record.name,
            "msg": record.getMessage(),
        }
        for k in ("trace_id","user_id","bucket_id","latency_ms"):
            if hasattr(record, k):
                base[k] = getattr(record, k)
        return json.dumps(base, ensure_ascii=False)

def setup_json_logging(level=logging.INFO):
    h = logging.StreamHandler(sys.stdout)
    h.setFormatter(JsonFormatter())
    root = logging.getLogger()
    root.handlers = [h]
    root.setLevel(level)
```

---

## Patch/Diff 제안 규칙
- **전체 파일 재작성 금지.** 변경은 **Unified Diff**로 제시한다.
- 예:
```diff
diff --git a/app/main.py b/app/main.py
--- a/app/main.py
+++ b/app/main.py
@@ -1,6 +1,12 @@
-from fastapi import FastAPI
+from fastapi import FastAPI, APIRouter
+router = APIRouter()

 app = FastAPI(title="Trend Helper API", version="0.1.0")

 @app.get("/health")
 def health():
     return {"ok": True}
+
+@router.post("/ideas")
+def ideas():
+    return {"titles": [], "tags": [], "script_beats": {}, "metadata": {}}
+app.include_router(router)
```

---

## Glossary (초보용)
- **Snapshot(스냅샷):** 특정 시점의 지표를 그대로 저장한 레코드(시간 경과 비교에 필요)  
- **Upsert:** 없으면 insert, 있으면 update(중복 방지)  
- **Velocity:** 조회수 증가 속도(분/분당 등), “지금 뜨는 중”을 판단  
- **UTC:** 전 세계 표준시간. 서버/DB/로그 시간을 한 곳에 맞추기 위함

---

## Parking Lot (TODO:V2)
- APScheduler 30–60분 주기 수집/증분  
- Google Trends/댓글, NLP 토큰화 + TF-IDF(+SVD) + KMeans  
- `/trending`, `/ideas/batch`, 인증/리밋/메트릭/알림, Redis 캐시

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dbdb1114) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
