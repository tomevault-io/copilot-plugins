## kakao-chat-summary

> > Claude Opus 4.6가 프로젝트를 이해하고 작업을 계속할 수 있도록 작성된 컨텍스트 파일입니다.

# 🤖 CLAUDE.md - AI 에이전트 프로젝트 컨텍스트

> Claude Opus 4.6가 프로젝트를 이해하고 작업을 계속할 수 있도록 작성된 컨텍스트 파일입니다.
> **새 대화 시작 시 `@CLAUDE.md`를 참조하세요.**

---

## 📋 프로젝트 개요

| 항목 | 값 |
|------|-----|
| **프로젝트명** | KakaoTalk Chat Summary |
| **목적** | 카카오톡 대화를 LLM으로 요약하고 관리하는 데스크톱 앱 |
| **언어** | Python 3.11+ |
| **GUI** | PySide6 (Qt for Python) |
| **DB** | SQLite + SQLAlchemy ORM |
| **버전** | v2.9.3 |
| **최종 업데이트** | 2026-04-24 |

---

## 🏗️ 프로젝트 구조

```
kakao-chat-summary/
├── src/
│   ├── app.py                 # 앱 진입점 (QApplication)
│   ├── ui/
│   │   ├── __init__.py
│   │   ├── main_window.py     # 메인 GUI (5000+ lines)
│   │   └── styles.py          # 카카오톡 스타일 테마
│   ├── db/
│   │   ├── __init__.py        # get_db() export
│   │   ├── database.py        # Database 클래스
│   │   └── models.py          # SQLAlchemy 모델 5개
│   ├── file_storage.py        # FileStorage 클래스
│   ├── full_config.py         # Config 클래스 (LLM 설정)
│   ├── parser.py              # KakaoLogParser 클래스
│   ├── detail_prompt.py       # 상세 분석 프롬프트 + HTML 템플릿 + LLM API
│   ├── url_extractor.py       # URL 추출 (마크다운 + HTML 파싱)
│   ├── import_to_db.py        # DB import 유틸
│   └── scheduler/
│       ├── __init__.py
│       └── tasks.py           # SyncScheduler (프레임워크 구현, 메인 앱 미연동)
├── data/
│   ├── db/                    # SQLite 데이터베이스
│   │   └── chat_history.db
│   ├── original/              # 원본 대화 (일별)
│   │   └── <채팅방>/
│   │       └── <채팅방>_YYYYMMDD_full.md
│   ├── summary/               # LLM 요약 (일별)
│   │   └── <채팅방>/
│   │       └── <채팅방>_YYYYMMDD_summary.md
│   ├── url/                   # URL 목록 (채팅방별 3개 파일)
│   │   └── <채팅방>/
│   │       ├── <채팅방>_urls_recent.md
│   │       ├── <채팅방>_urls_weekly.md
│   │       └── <채팅방>_urls_all.md
│   └── detail_summary/        # 상세 분석 HTML (v2.8.0)
│       └── <채팅방>/
│           └── <채팅방>_YYYYMMDD_detail.html
├── output/                    # CLI 스크립트 (src/manual/) 출력 디렉터리
├── upload/                    # 파일 업로드 기본 디렉터리
├── logs/                      # 로그 (summarizer_YYYYMMDD.log)
├── docs/                      # 문서 (01-prd ~ 06-tasks)
├── .env.local                 # API 키 (gitignore)
├── env.local.example          # API 키 예제
├── requirements.txt
├── .gitignore
├── README.md
└── CLAUDE.md                  # 이 파일
```

---

## 🗃️ 데이터베이스 스키마 (5개 테이블)

### ChatRoom
```python
class ChatRoom(Base):
    __tablename__ = 'chat_rooms'
    id = Column(Integer, primary_key=True)
    name = Column(String(255), nullable=False)
    file_path = Column(String(512))
    participant_count = Column(Integer, default=0)
    last_sync_at = Column(DateTime)
    created_at = Column(DateTime, default=datetime.now)
    # Relationships: messages, summaries, sync_logs, urls
```

### Message
```python
class Message(Base):
    __tablename__ = 'messages'
    id = Column(Integer, primary_key=True)
    room_id = Column(Integer, ForeignKey('chat_rooms.id'))
    sender = Column(String(255), nullable=False)
    content = Column(Text)
    message_date = Column(Date, nullable=False)
    message_time = Column(Time)
    raw_line = Column(Text)
    created_at = Column(DateTime)
    # UniqueConstraint: (room_id, sender, message_date, message_time, content)
```

### Summary
```python
class Summary(Base):
    __tablename__ = 'summaries'
    id = Column(Integer, primary_key=True)
    room_id = Column(Integer, ForeignKey('chat_rooms.id'))
    summary_date = Column(Date, nullable=False)
    summary_type = Column(String(50))  # 'daily', '2days', 'weekly'
    content = Column(Text)
    llm_provider = Column(String(100))
    token_count = Column(Integer)
    created_at = Column(DateTime)
```

### SyncLog
```python
class SyncLog(Base):
    __tablename__ = 'sync_logs'
    id = Column(Integer, primary_key=True)
    room_id = Column(Integer, ForeignKey('chat_rooms.id'))
    status = Column(String(50))  # 'success', 'failed', 'partial'
    message_count = Column(Integer)
    new_message_count = Column(Integer)
    error_message = Column(Text)
    synced_at = Column(DateTime)
```

### URL
```python
class URL(Base):
    __tablename__ = 'urls'
    id = Column(Integer, primary_key=True)
    room_id = Column(Integer, ForeignKey('chat_rooms.id'))
    url = Column(Text, nullable=False)
    descriptions = Column(Text)  # " / " 구분자
    source_date = Column(Date)
    created_at = Column(DateTime)
    updated_at = Column(DateTime)
    # UniqueConstraint: (room_id, url)
```

---

## 🖥️ GUI 구조 (main_window.py)

### 메인 윈도우 레이아웃
```
┌─────────────────────────────────────────────────────────┐
│ 메뉴바                                                   │
│ ├─ 파일: 채팅방 추가, 채팅방 삭제, 종료                   │
│ ├─ 도구: 상세 분석 생성, 전체 채팅방 상세분석/URL,          │
│ │       백업/복원, DB 복구, 채팅방 복구, 설정               │
│ └─ 도움말: 정보                                          │
├──────────────┬──────────────────────────────────────────┤
│              │  QTabWidget (4개 탭)                      │
│  채팅방 목록  │  ├─ 📊 대시보드                           │
│  (QListWidget)│  ├─ 📅 날짜별 상세 분석                    │
│              │  ├─ 🔗 URL 정보 (동기화/복구 버튼 포함)    │
│              │  └─ 🔧 기타 (통계 갱신 등)                │
│              │                                          │
│ [➕ 채팅방]   │                                          │
│ [📤 업로드]  │                                          │
├──────────────┴──────────────────────────────────────────┤
│ 상태바: [아이콘] 메시지  [🤖 요약 중 3/10 ▮▮▯ ❌] (HH:MM:SS) │
└─────────────────────────────────────────────────────────┘
```

### 다이얼로그 클래스 (5개)
| 클래스 | 역할 |
|--------|------|
| `CreateRoomDialog` | 채팅방 생성 |
| `UploadFileDialog` | 파일 업로드 (기본 디렉터리: upload/) |
| `SummaryProgressDialog` | 진행률 다이얼로그 (보존) |
| `SettingsDialog` | 설정 |
| (QMessageBox) | 각종 알림 |

### 위젯 클래스
| 클래스 | 역할 |
|--------|------|
| `ChatRoomWidget` | 채팅방 목록 아이템 |
| `DashboardCard` | 대시보드 통계 카드 (`update_card()` 메서드) |
| `SummaryProgressWidget` | 상태바 내장 비모달 프로그레스 (아이콘+메시지+프로그레스바+취소) |

### Worker 스레드 (6개)
| 클래스 | 역할 |
|--------|------|
| `FileUploadWorker` | 파일 업로드 및 파싱 |
| `AllRoomsUrlSyncWorker` | 전체 채팅방 URL 동기화 (상세 분석 HTML에서 추출) |
| `DetailSummaryWorker` | 단일 날짜 상세 분석 HTML 생성 |
| `DetailBatchWorker` | 채팅방 내 여러 날짜 상세 분석 일괄 생성 |
| `AllRoomsDetailWorker` | 전체 채팅방 상세 분석 일괄 생성 |
| `RecoveryWorker` | 파일에서 DB 복구 |

### 상태바 아이콘
| 아이콘 | 의미 |
|--------|------|
| ✅ | 성공/준비 완료 |
| ⏳ | 작업 진행 중 |
| ❌ | 실패 |
| ⚠️ | 경고 |
| ℹ️ | 정보 |

---

## 🔑 핵심 기능

### 1. 채팅방 관리
- 채팅방 생성 (Enter 키로 즉시 생성 가능)
- 채팅방 삭제 (파일 메뉴 → 현재 선택된 채팅방 삭제, 확인 다이얼로그)
- 채팅방 목록 (메시지 개수 내림차순 정렬)
- 파일 업로드 (기본 디렉터리: `upload/`)

### 2. LLM 상세 분석 생성 (v2.9.0: 기본 요약 제거, 상세 분석 전용)
- **지원 LLM**: Z.AI GLM (glm-4.5), OpenAI GPT-4o-mini, MiniMax (M2.7), Perplexity (sonar), Grok, OpenRouter, Kilo, Ollama
  - *제약사항*: 컨텍스트가 작은 모델은 `max_input_chars`/`max_tokens` 안전 가드 적용. 대화량이 방대한 방은 GLM이나 MiniMax 사용 권장.
- **분석 옵션**: 분석 필요한 날짜만 (기본), 오늘, 어제~오늘, 엇그제~오늘, 전체 일자 + 이미 분석된 날짜 건너뛰기 체크박스
- **응답 검증**: finish_reason, 최소 길이, 필수 섹션, 잘림 패턴
- **추론 내용 제거**: LLM의 `<think>` 태그 및 본문 이전 추론 텍스트 자동 제거
- **한자/일본어 후처리** (v2.8.2): `hanja` 라이브러리로 한자→한글 독음 변환, 일본어 자동 제거
- **진행 상황**: 실시간 진행률, 취소 가능

### 3. 상세 분석 HTML (유일한 요약 경로, v2.9.0)
- **프롬프트** (`detail_prompt.py`): 토픽별 심층 분석, URL 모음, 감정/온도 분석, 핵심 시사점
- **다크 테마 HTML**: `data/detail_summary/`에 저장, 브라우저에서 열기 지원
- **생성 방식 3가지**:
  - 날짜 탭에서 단일 날짜 상세 생성 (`DetailSummaryWorker`)
  - 날짜 탭에서 일괄 상세 생성 (`DetailBatchWorker`)
  - 도구 메뉴에서 전체 채팅방 상세 분석 (`AllRoomsDetailWorker`, Ctrl+Shift+G)
- **URL 처리**: 대화 내 모든 URL을 토픽별 근거 + "🔗 공유된 URL 모음" 섹션에 설명과 함께 정리

### 4. 대시보드 탭
- 채팅방 통계 (메시지 수, 참여자 수)
- 채팅방 정보 표시

### 5. 날짜별 상세 분석 탭
- 달력 위젯으로 날짜 선택 (QCalendarWidget)
- 상세 분석 HTML 렌더링 + [🔍 상세 생성] [📂 브라우저] [🔍 일괄 상세] 버튼

### 6. URL 정보 탭
- **3개 섹션**: 최근 3일 (50개 제한), 최근 1주 (무제한), 전체 (무제한)
- **URL 추출**: 상세 분석 HTML의 `<a href>` + `url-card` 구조에서 파싱 (v2.9.0)
- **URL 정규화**: 특수문자, fragment, trailing slash 제거
- **중복 제거**: `deduplicate_urls()` 함수
- **동기화/복구 버튼**: 상세 분석에서 URL 추출, 파일에서 DB 복구

### 7. 파일 기반 저장
- `data/original/`: 원본 대화 (일별 MD)
- `data/detail_summary/`: 상세 분석 HTML
- `data/url/`: URL 목록 (채팅방별 3개 파일)
- `data/summary/`: (레거시, v2.8.x 이전 기본 요약. 읽기만 가능, 신규 생성 안 함)

### 8. 기타 기능 탭
- **📊 통계 정보 갱신**: 대시보드 통계와 채팅방 목록을 최신 상태로 갱신
- **💾 채팅방 백업**: 선택된 채팅방의 파일을 백업 (도구 메뉴와 동일 기능)
- **📂 채팅방 복원**: 백업에서 특정 채팅방 복원 (현재 선택된 채팅방이 기본 선택됨)

### 9. 도구 메뉴 복구 기능
- **🗄️ DB 전체 복구**: `data/original/` + `data/detail_summary/` + `data/url/` → DB 재구축 (파괴적)
- **💬 채팅방 복구**: 파일 디렉터리에 있지만 DB에 없는 채팅방만 추가 (비파괴적)

---

## 📁 주요 모듈 상세

### src/db/database.py - Database 클래스
```python
# 채팅방
create_room(name, file_path=None) -> ChatRoom
get_all_rooms() -> List[ChatRoom]  # 메시지 수 내림차순
get_room_by_id(room_id) -> Optional[ChatRoom]
get_room_by_name(name) -> Optional[ChatRoom]
get_room_stats(room_id) -> Dict[str, Any]
update_room_sync_time(room_id)
delete_room(room_id)

# 메시지
add_messages(room_id, messages, batch_size=500) -> int  # 배치, 중복 체크
get_messages_by_room(room_id, start_date=None, end_date=None) -> List[Message]
get_message_count_by_room(room_id) -> int
get_message_count_by_date(room_id, target_date) -> int
get_unique_senders(room_id) -> List[str]

# 요약
add_summary(room_id, summary_date, summary_type, content, llm_provider=None) -> Summary
get_summary_by_id(summary_id) -> Optional[Summary]
get_summaries_by_room(room_id, summary_type=None) -> List[Summary]
delete_summary(room_id, summary_date) -> bool

# 동기화 로그
add_sync_log(room_id, status, message_count, new_message_count, error_message) -> SyncLog
get_sync_logs_by_room(room_id, limit=10) -> List[SyncLog]

# URL
add_url(room_id, url, descriptions=None, source_date=None) -> URL
add_urls_batch(room_id, urls) -> int
get_urls_by_room(room_id) -> Dict[str, List[str]]
get_url_count_by_room(room_id) -> int
clear_urls_by_room(room_id) -> int
```

### src/file_storage.py - FileStorage 클래스
```python
# 디렉터리
base_dir = Path("data")
original_dir = base_dir / "original"
summary_dir = base_dir / "summary"
url_dir = base_dir / "url"

# 원본 대화
save_daily_original(room_name, date_str, messages) -> Path
save_all_daily_originals(room_name, messages_by_date, cutoff_date=None) -> List[Path]  # cutoff 미만 과거 보호
load_daily_original(room_name, date_str) -> List[str]
get_available_dates(room_name) -> List[str]

# 요약
save_daily_summary(room_name, date_str, content, llm) -> Path
load_daily_summary(room_name, date_str) -> str
get_summarized_dates(room_name) -> List[str]
get_dates_needing_summary(room_name) -> Dict[str, str]  # 마지막 요약일(포함) 이후만 반환 ("new"/"resummary")
invalidate_summary_if_content_changed(room_name, date_str, old_hash, new_hash, old_count, new_count, threshold=10)  # 최근 날짜만, +10개 이상 시 무효화

# URL (3개 파일)
save_url_lists(room_name, urls_recent, urls_weekly, urls_all)
load_url_list(room_name, list_type) -> Dict[str, List[str]]
```

### src/url_extractor.py - URL 추출 함수
```python
extract_urls_from_text(text, section_only=False) -> Dict[str, List[str]]
extract_url_with_description(line) -> Tuple[str, str]
normalize_url(url) -> str  # 특수문자, fragment, trailing slash 제거
deduplicate_urls(urls_dict) -> Dict[str, List[str]]
save_urls_to_file(urls_dict, filepath) -> bool
```

### src/full_config.py - Config 클래스
```python
# LLM 제공자
LLM_PROVIDERS = {
    "glm": {..., env_key="ZAI_API_KEY"},
    "chatgpt": {..., env_key="OPENAI_API_KEY"},
    "minimax": {..., env_key="MINIMAX_API_KEY"},
    "perplexity": {..., env_key="PERPLEXITY_API_KEY"}
}

get_api_key(provider) -> str
set_api_key(api_key, provider)
```

---

## 🔧 환경 설정

### .env.local 형식
```bash
ZAI_API_KEY=your_key_here
OPENAI_API_KEY=your_key_here
MINIMAX_API_KEY=your_key_here
PERPLEXITY_API_KEY=your_key_here
```

---

## 🚀 실행 방법

```bash
# GUI 앱 실행
python src/app.py
```

> **참고**: v2.9.0에서 CLI 스크립트(`src/manual/`)와 기본 요약 모듈(`llm_client.py`, `chat_processor.py`)은 제거되었습니다. 모든 요약은 상세 분석(HTML)으로 통합되었습니다.

---

## ⚠️ 주의사항

1. **API 키**: `.env.local`은 절대 커밋하지 않음
2. **데이터**: `data/` 폴더는 `.gitignore`에 포함
3. **한글 인코딩**: 파일 읽기/쓰기 시 `encoding='utf-8'` 필수
4. **Qt 스레드**: UI 업데이트는 메인 스레드에서만 (Signal/Slot)
5. **PowerShell**: `&&` 대신 명령어 분리 실행

---

## 🔮 향후 개선 사항 (Pending)

1. [ ] APScheduler 메인 앱 연동 (SyncScheduler 프레임워크는 구현 완료)
2. [ ] 새 메시지 개수 표시
3. [ ] 설정 다이얼로그에서 API 키 입력
4. [ ] 요약 품질 평가 기능
5. [ ] 테스트 코드 작성 (pytest)

---

## 🐛 트러블슈팅 히스토리

### v2.2.2 - 요약 파일 ↔ DB 동기화 버그 (2026-02-01)

**증상**: LLM 요약 생성 후 대시보드에 요약이 표시되지 않음

**원인 3가지**:
1. **SummaryGeneratorWorker가 파일에만 저장** (`main_window.py` ~826행)
   - `storage.save_daily_summary()`만 호출하고 `db.add_summary()`는 호출하지 않았음
   - 대시보드는 DB에서 요약 목록을 읽으므로 불일치 발생
2. **RecoveryWorker date 타입 버그** (`main_window.py` ~916행)
   - `db.add_summary()`에 `date_str` (문자열)을 전달 → `date` 객체 필요
3. **RecoveryWorker 요약 내용 500자 잘림** (`main_window.py` ~920행)
   - `summary_content[:500]`으로 잘라서 저장 → 전체 내용 손실

**수정**:
1. SummaryGeneratorWorker에서 파일 저장 직후 `db.delete_summary()` + `db.add_summary()` 호출 추가
2. RecoveryWorker에서 `datetime.strptime(date_str, '%Y-%m-%d').date()` 변환 적용
3. `summary_content[:500]` → `summary_content` 전체 저장으로 변경
4. `database.py`에 `delete_summary(room_id, summary_date)` 메서드 신규 추가

**설계 원칙**: 요약 필요 여부는 **파일 존재 여부**로 판단 (`get_dates_needing_summary`, `get_summarized_dates`).
DB에 데이터가 있어도 파일이 없으면 재수집 대상이며, DB 저장 시 기존 행을 삭제 후 추가하여 중복 방지.

### v2.2.3 - UI 개선 및 중복 헤더 제거 (2026-02-01)

**변경 1: 채팅방 삭제를 파일 메뉴로 이동**
- ChatRoomWidget의 ✕ 삭제 버튼 제거 (오클릭 위험, 버튼이 보이지 않는 문제)
- 파일 메뉴에 "채팅방 삭제..." 액션 추가
- 현재 선택된 채팅방(`self.current_room_id`)을 기준으로 삭제
- 채팅방 미선택 시 경고 메시지 표시

**변경 2: CreateRoomDialog Enter 키 수정**
- Enter 키가 취소(reject) 대신 만들기(accept) 동작하도록 수정
- `name_input.returnPressed` → `_on_create` 연결
- `create_btn.setDefault(True)` 설정
- 빈 이름 입력 시 가드 추가

**변경 3: 요약 헤더 중복 제거**
- `chat_processor._format_as_markdown()`의 "카카오톡 대화 요약 리포트" 헤더/푸터 제거
- `file_storage._format_summary_content()`의 헤더만 사용 (채팅방명, 날짜, LLM, 생성 시각 포함)

**변경 4: placeholder 개인정보 제거**
- CreateRoomDialog의 placeholder를 실제 고객명에서 일반 그룹명으로 변경

**변경 5: LLM read timeout 증가**
- `llm_client.py`의 read_timeout을 300초 → 600초로 증가

### v2.3.0 - 비모달 요약 프로그레스 & 대시보드 개선 (2026-02-01)

**변경 1: 모달 다이얼로그 → 상태바 프로그레스 위젯**
- `SummaryProgressDialog` (모달) 대신 `SummaryProgressWidget` (비모달) 사용
- 상태바에 `[채팅방명] LLM 요약 중...` + 프로그레스바 + 취소 버튼 표시
- 요약 중 메인 윈도우 자유롭게 조작 가능 (다른 채팅방 조회 등)

**변경 2: 중복 실행 방지 & 채팅방 전환 처리**
- `_summary_in_progress` 플래그로 중복 실행 차단
- `summary_source_room_id`로 요약 대상 채팅방 추적
- 완료 시: 같은 방이면 대시보드 갱신, 다른 방이면 상태바 알림만

**변경 3: closeEvent 추가**
- 요약 진행 중 앱 종료 시 확인 다이얼로그 → worker.cancel() + wait(5000)

**변경 4: 대시보드 카드 컴팩트화**
- `DashboardCard`에 `update_card(value, subtext)` 메서드 추가
- 아이콘+제목+값을 한 줄로 배치 (기존 3줄 → 1줄 + 서브텍스트)
- 폰트/패딩 축소로 카드 높이 절반 감소
- 서브텍스트에 실제 통계 표시: 대화 기간, 요약 진행률(N/M일, %)

**변경 5: DetachedInstanceError 수정**
- `database.py` sessionmaker에 `expire_on_commit=False` 추가
- 세션 종료 후 ORM 객체 속성 접근 시 에러 발생하던 버그 수정

### v2.3.1 - 기타 탭, 채팅방 복구, 도움말 정보 (2026-02-02)

**변경 1: 4번째 탭 "🔧 기타" 추가**
- 통계 정보 갱신 카드 (대시보드 + 채팅방 목록 새로고침)
- 향후 기능 추가 영역으로 활용

**변경 2: 도구 메뉴에 복구 기능 배치**
- DB 전체 복구 (`_on_recovery`): 파괴적 복구
- 채팅방 복구 (`_on_room_recovery`): 비파괴적, 파일 디렉터리 스캔 후 누락된 채팅방만 추가

**변경 3: URL 탭 동기화/복구 버튼 유지**
- URL 탭 헤더에 동기화/파일 복구 버튼 배치 (채팅방별 기능이므로 URL 탭에 유지)

**변경 4: `FileStorage.get_all_rooms()` url 디렉터리 포함**
- 기존: `data/original/` + `data/summary/`만 스캔
- 변경: `data/url/` 디렉터리도 포함하여 누락 방지

**변경 5: 도움말 정보 업데이트**
- 버전 2.3.1, 제작자: 민연홍, GitHub 링크 추가

---

## 📚 관련 문서

| 파일 | 내용 |
|------|------|
| `docs/01-prd.md` | 제품 요구사항 |
| `docs/02-trd.md` | 기술 요구사항 |
| `docs/03-user-flow.md` | 사용자 흐름 |
| `docs/04-data-design.md` | 데이터 설계 |
| `docs/05-coding-convention.md` | 코딩 컨벤션 |
| `docs/06-tasks.md` | 작업 목록 & 버전 히스토리 |

---

### v2.4.0 - 데이터 안전성 및 견고성 강화 (2026-02-04)

**이슈 1: 채팅 데이터 로드 시 기존 데이터 삭제 (Data Loss)**
- **증상**: 채팅 파일 재업로드 시 기존 데이터가 사라지는 현상
- **원인**: `save_daily_original`에서 파싱 실패(0건) 또는 단순 병합 시 데이터 감소(파싱 포맷 차이 등)를 체크하지 않고 덮어씀
- **수정**:
  - `file_storage.py`: 기존 파일이 존재하는데 파싱 결과가 0건이면 원본 내용을 read_text로 복구
  - 병합 결과가 기존 데이터보다 적으면(`merged < existing`) 저장을 건너뛰는(Skip) 안전 장치 추가

**이슈 2: 요약 파일 대량 삭제 (Summary Invalidation)**
- **증상**: 업로드 시 모든 날짜의 요약 파일이 삭제됨
- **원인**: 메시지 데이터가 소량 변경(+1건)되거나 이전 불완전 데이터(134건)가 완전한 데이터(452건)로 바뀔 때 무조건 `New > Old` 조건으로 삭제(`unlink`)
- **수정**:
  - `file_storage.py`: `delete_daily_summary`를 `.bak` 파일로 리네임하는 백업 로직으로 변경 (영구 삭제 방지)

**이슈 3: DB 파일 손상 (Database Corruption) - 근본 원인 수정**
- **증상**: `sqlalchemy.exc.DatabaseError: database disk image is malformed` 발생 (LLM 요약 중 특히 빈번)
- **근본 원인**: **워커 스레드들이 싱글톤 `get_db()` 인스턴스를 공유**
  - `FileUploadWorker`, `SyncWorker`, `SummaryGeneratorWorker`가 모두 `__init__`에서 `self.db = get_db()`로 동일한 DB 인스턴스를 가져감
  - 워커 스레드가 DB 쓰기(`add_messages`, `add_summary`)를 하는 동안, 메인 UI 스레드도 DB 읽기(`get_all_rooms`, `get_room_stats`)를 수행
  - SQLite는 **단일 쓰기자(Single Writer)** 모델이므로, 동시 접근 시 `SQLITE_BUSY` 또는 저널 파일 충돌로 DB 손상 발생
- **수정**:
  - `main_window.py`: 각 워커 클래스의 `run()` 메서드 내에서 **전용 `Database()` 인스턴스**를 생성하도록 변경
  - 작업 완료 후 `worker_db.engine.dispose()`로 연결을 명시적으로 해제
  - 수정된 워커: `FileUploadWorker`, `SyncWorker`, `SummaryGeneratorWorker`
- **추가 방어**:
  - `llm_client.py`: API 500/Network 에러 시 최대 3회 재시도(Exponential Backoff) 로직 추가
  - `database.py`: `add_sync_log` 등 주요 쓰기 메서드에 `try-except` 블록을 씌워 로깅 실패가 앱 충돌로 이어지지 않도록 방어

**이슈 4: 앱 강제 종료 (App Crash)**
- **증상**: 요약 생성 도중 또는 파일 업로드 중 앱이 응답 없음 상태가 되거나 강제 종료됨 (특히 MiniMax 모델 사용 시에도 발생)
- **원인**: 디버깅을 위해 추가한 `print()` 문이 반복문 내에서 수천 번 실행되면서 콘솔 출력 버퍼 오버플로우 또는 Windows 콘솔 I/O 블로킹 유발 (UI 스레드와 워커 스레드 간 간섭 심화)
- **수정**:
  - `main_window.py`: 반복문 내의 디버그용 `print()` 구문 전면 제거

**이슈 5: 파일 재업로드 시 모든 요약 삭제 (Mass Invalidation)**
- **증상**: 파일을 재업로드하면 오늘만이 아니라 **모든 날짜**의 요약이 무효화됨
- **원인**: 메시지 수 비교 방식의 부정확성 (포맷 차이로 인한 오감지)
- **수정 (v2.4.0)**: 파일 크기 비교 방식으로 변경
  - `file_storage.py`: `get_original_file_size()`, `invalidate_summary_if_file_changed()` 추가
  - `main_window.py`: `FileUploadWorker`가 저장 전/후 파일 크기를 비교하여 실제 변경된 날짜만 무효화
- **재수정 (v2.5.1)**: **메시지 내용 해시 비교 방식**으로 변경
  - **잔존 문제**: `_format_original_content()`의 헤더에 `저장 시각`이 포함되어 있어, 동일한 메시지를 재업로드해도 파일 크기가 미세하게 변동 → 불필요한 요약 무효화 발생
  - `file_storage.py`: `get_original_content_hash()` 추가 — 헤더/푸터 제외, 메시지 본문만 MD5 해시
  - `file_storage.py`: `invalidate_summary_if_content_changed()` 추가 — 해시 비교로 무효화 판단
  - `file_storage.py`: 기존 `invalidate_summary_if_file_changed()` deprecated 처리
  - `main_window.py`: `FileUploadWorker`가 파일 크기 대신 메시지 해시로 비교
  - **효과**: 메시지 내용이 동일하면 저장 시각이 바뀌어도 요약 유지 ✅

---

### v2.5.2 - 임계값 기반 요약 무효화 (2026-02-26)

**이슈: 사용자 출입으로 인한 불필요한 요약 재수행**
- **증상**: 카카오톡 대화를 전체 내보내기할 때마다 사용자가 나간/들어온 날짜에 시스템 메시지(1-2개)가 추가되어 메시지 해시가 달라짐 → 불필요한 요약 무효화 발생
- **원인**: v2.5.1의 메시지 해시 비교 방식은 1개 메시지 차이도 감지하여 무조건 요약 무효화
- **근본 원인 분석**:
  - 카카오톡은 **처음부터 끝까지 전체 대화**를 내보냄 (증분 백업 아님)
  - 사용자가 나가면 해당 날짜에 `[사용자]님이 나갔습니다` 시스템 메시지 추가
  - 대부분의 날짜는 메시지 개수가 동일하지만, 사용자 출입이 있는 날짜만 1-2개 증가
  - 실제 대화가 추가되는 경우는 하루 50개 이상 변경이 거의 없음

**해결 방안: 메시지 개수 변경 임계값 도입**
- **임계값 50개**: 메시지 개수 변경이 50개 미만이면 요약 유지, 50개 이상이면 무효화
- **50개 미만 변경 (무시)**:
  - 사용자 나가기/들어오기: +1~3개
  - 시스템 메시지 몇 개: +5~10개
  - 짧은 대화 추가: +30개
  → 요약 유지 ✅
- **50개 이상 변경 (재수집)**:
  - 실제 하루치 대화 추가: +80~200개
  - 부분 업로드/데이터 손상: -60개 이상
  → 요약 무효화 🔄

**수정 내역**:
1. `file_storage.py` - `invalidate_summary_if_content_changed()` 메서드 개선
   - 파라미터 추가: `old_count`, `new_count`, `threshold=50`
   - 메시지 개수 차이를 절댓값으로 계산: `abs_diff = abs(new_count - old_count)`
   - 임계값 미만이면 해시가 달라도 요약 유지
   - 임계값 이상이면 요약 무효화 (증가/감소 모두 처리)
   - 로그 개선:
     - `ℹ️ [날짜] 메시지 +2개 (< 50개) → 요약 유지`
     - `🔄 [날짜] 메시지 +80개 (≥ 50개, 대량 추가) → 요약 무효화`
     - `🔄 [날짜] 메시지 -60개 (≥ 50개, ⚠️ 데이터 감소) → 요약 무효화`

2. `main_window.py` - `FileUploadWorker.run()` 메서드 개선
   - 저장 전 메시지 개수 추적: `old_message_counts[date_str]`
   - 저장 후 메시지 개수 계산: `new_count`
   - `invalidate_summary_if_content_changed()` 호출 시 개수 정보 전달

**효과**:
- 사용자 출입으로 인한 불필요한 재요약 제거 ✅
- 실제 대화가 추가된 경우만 재요약 수행 ✅
- 부분 업로드/데이터 손상 감지 및 경고 ✅

---

### v2.6.0 - 과거 데이터 보호 및 백업/복원 UI 개선 (2026-03-15)

**이슈 1: 업로드 시 과거 요약 대량 삭제**
- **증상**: 채팅방에서 사용자가 나가면서 시스템 메시지가 추가 → 재업로드 시 과거 날짜의 요약까지 무효화
- **근본 원인**: v2.5.2의 임계값(50개)은 최근 날짜에만 필요하지만, 모든 날짜에 적용되어 과거 요약도 위험에 노출
- **수정 — cutoff 기반 과거 보호 정책**:
  - **마지막 요약일-1일 이전**: 원본 파일, 해시 계산, DB 저장, 요약 무효화 모두 건너뜀 (완전 보호)
  - **마지막 요약일-1일 이후**: 원본 파일 저장/merge, 메시지 +10개 이상 추가 시에만 무효화
  - `file_storage.py`: `save_all_daily_originals()`에 `cutoff_date` 파라미터 추가
  - `file_storage.py`: `invalidate_summary_if_content_changed()` 임계값 50 → 10, cutoff 판단 제거 (호출자가 필터링)
  - `main_window.py`: `FileUploadWorker`가 cutoff 계산 후 최근 날짜만 처리

**이슈 2: 요약 필요 날짜 카운트 과다**
- **증상**: 요약 생성 다이얼로그에서 "40일 요약 필요"로 표시되지만 실제로는 15일만 필요
- **원인**: `get_dates_needing_summary()`가 요약 파일 없는 **모든** 날짜를 반환 (과거 무효화된 날짜 포함)
- **수정**:
  - `file_storage.py`: `get_dates_needing_summary()` — 마지막 요약일(포함) 이후만 반환
  - 마지막 요약일은 `"resummary"`, 이후 미요약 날짜는 `"new"`로 구분
  - 과거 미요약 날짜는 "전체 일자" 옵션으로 수동 요약 가능

**이슈 3: 백업/복원 메뉴 구조 개선**
- **변경 전**: "📂 백업에서 복원..." 하나에 전체/개별 복원이 혼합
- **변경 후**:
  - 도구 메뉴: "📂 전체 백업에서 복원..." + "📂 채팅방 복원..." 분리
  - 기타 탭: 채팅방 백업 카드 (주황색) + 채팅방 복원 카드 (초록색) 추가
  - 기타 탭 복원 시 현재 선택된 채팅방이 기본 선택됨

---

## ⚠️ 개발 시 주의사항

### 코드 변경 후 재기동
- **LLM 요약 수집 중일 때는 앱 종료 전 요약 완료를 기다려야 함**
- `closeEvent`에서 요약 진행 중이면 확인 다이얼로그를 표시하고, `worker.cancel()` + `wait(5000)`으로 안전하게 종료
- 요약 중간에 강제 종료하면 일부 날짜의 요약이 유실될 수 있음

### 백업/복원 기능 (v2.5.0)

**도구 메뉴 구조 (v2.9.0):**
```
도구
├── 🔍 상세 분석 생성 (Ctrl+G)
├── 🌐 전체 채팅방 상세 분석 생성 (Ctrl+Shift+G)
├── 🌐 전체 채팅방 URL 동기화 (Ctrl+Shift+U)
├── ─────────────
├── 💾 전체 백업... (Ctrl+B)     - DB + 모든 파일 백업
├── 💾 채팅방 백업...            - 선택된 채팅방만 백업
├── ─────────────
├── 📂 전체 백업에서 복원...     - 백업에서 전체 복원 (DB 포함)
├── 📂 채팅방 복원...            - 백업에서 특정 채팅방만 복원
├── ─────────────
├── 🔄 파일에서 DB 재구축...     - 현재 파일에서 DB 재생성 (파괴적)
├── 🔄 누락 채팅방 DB 추가...    - 파일에는 있지만 DB에 없는 채팅방 추가
```

**용어 정리:**
- **백업/복원**: `data/backup/` 스냅샷 관리
- **재구축/DB 추가**: 현재 `data/original/`, `data/detail_summary/` ↔ DB 동기화

---

### v2.7.0 - 전체 채팅방 일괄 처리 (2026-03-28)

**기능 1: 전체 채팅방 LLM 요약 생성**
- 도구 메뉴 → `🌐 전체 채팅방 LLM 요약 생성` (Ctrl+Shift+G)
- `AllRoomsSummaryOptionsDialog`: 채팅방별 요약 현황 스크롤 리스트, LLM 선택, 범위 선택 (pending/today/all 등)
- `AllRoomsSummaryWorker`: 모든 채팅방을 순회하며 날짜별 요약 생성 (스레드 안전, 취소 지원)
- 상태바 프로그레스 위젯으로 실시간 진행률 표시
- 채팅방별 결과 요약 다이얼로그 표시 (✅/⚠️/⏭️ 아이콘)

**기능 2: 전체 채팅방 URL 동기화**
- 도구 메뉴 → `🌐 전체 채팅방 URL 동기화` (Ctrl+Shift+U)
- `AllRoomsUrlSyncWorker`: 모든 채팅방의 요약에서 URL 추출 → DB + 파일(3개: recent/weekly/all) 저장
- 확인 다이얼로그에서 대상 채팅방 목록 표시 후 실행
- 완료 시 현재 채팅방 URL 탭 자동 갱신

**closeEvent 개선**: 전체 채팅방 워커(요약/URL) 실행 중 앱 종료 시에도 안전 종료 처리

---

### v2.8.5 - 백그라운드 트레이 연동 및 한글/토큰 한계 대응 (2026-04-12)
- **트레이 아이콘 연동**: 백그라운드 구동 지속성 확보
- **한글 인코딩 및 토큰 초과 방어**: `.env.local` 파서 수정 (`max_input_chars` 적용)

### v2.8.4 - DeepSeek 컨텍스트 제한 대응 및 프롬프트 개선 (2026-04-09)
- 🔧 **DeepSeek 32K 컨텍스트 한계 대응**: `qwen-or` 및 `qwen-kilo` 제공자에 대해 입력 문자열을 40,000자로 강제 잘라내어 API 400 에러를 방지 (`max_input_chars` 필드 추가) 및 출력 토큰을 8000으로 제한.
- 🔧 **상세 분석 토픽 추출량 상향**: 상세 분석 프롬프트의 토픽 추출 개수 제약을 '최소 10개 이상'에서 '최소 20개 이상(많으면 30~40개)'으로 상향하여 보다 심층적인 요소를 도출하도록 개선.
- 🐛 **URL 자동 동기화 버그 수정**: `main_window.py` 내 `_auto_sync_urls` 함수에서 발생하던 `NameError: name 'logger' is not defined` 버그 수정.

### v2.8.3 - 상세 분석 앱 렌더링 보정 (2026-04-08)
- `QTextBrowser`에서 잘못된 닫힘 태그(`</hp>` 등)로 인해 제목 스타일이 본문까지 번지는 문제 수정

### v2.8.2 - URL 정보 강화 및 한자/일본어 후처리 (2026-04-05)

- **한자→한글 독음 자동 변환**: `hanja` 라이브러리를 활용하여 LLM 응답의 한자를 한글 독음으로 변환 (예: 推荐→추천, 新闻→신문), 일본어(히라가나/가타카나) 자동 제거. 기본 요약(`llm_client.py`)과 상세 분석(`detail_prompt.py`) 모두 적용.
- **URL 정보 포맷 강화**: 기본 요약의 링크/URL 섹션에 내용/시사점/활용 구조 추가 (상세 분석과 동일한 형식).
- **URL 추출기 멀티라인 지원**: `url_extractor.py`가 새 멀티라인 URL 포맷(제목, 내용, 시사점, 활용)을 파싱하도록 개선. 기존 한 줄 포맷도 호환.
- **URL 탭 표시 개선**: URL 정보 탭에서 내용/시사점/활용을 구조적으로 렌더링.
- **LLM 선택 동적 모델명**: UI의 LLM 선택 드롭다운이 `full_config.py`의 모델명을 동적으로 표시 (하드코딩 제거).
- **LLM 모델 변경**: GLM `glm-5-turbo` → `glm-4.5`, MiniMax `MiniMax-M2.5` → `MiniMax-M2.7`.
- **의존성 추가**: `hanja>=0.15.0` (requirements.txt).

### v2.8.1 - 프롬프트 제약 및 무효화 로직 개편 (2026-04-03)

- **LLM 시스템 프롬프트 규칙 강화**: LLM 요약 시 중국어, 일본어, 아랍어 등 불필요한 외국어로 응답하는 현상을 방지하도록 한글 설명/영어 용어 사용 강제 제약 추가.
- **무효화 로직 개편**: 파일 업로드 시 대화 파일에 변동(10개 이상)이 있을 경우 호출되는 무효화 로직 수행 과정에서, 기존의 `summary` 뿐만 아니라 새롭게 도입된 `detail_summary`의 HTML 파일도 함께 `.bak`으로 백업하고 새롭게 재생성하도록 로직 수정.

### v2.8.0 - 상세 분석 HTML 생성 기능 (2026-03-29)

**기능 1: 상세 분석 HTML 생성**
- 기존 기본 요약과 별도로, 토픽별 심층 분석 HTML을 생성하는 기능 추가
- `src/detail_prompt.py`: 상세 분석 전용 프롬프트 + 다크 테마 HTML 템플릿 + LLM API 호출
- `data/detail_summary/<채팅방>/<채팅방>_YYYYMMDD_detail.html` 파일로 저장
- 프롬프트 구조: 키워드 TOP5 → 토픽별 분석(4~7개) → URL 모음 → 감정/온도 분석 → 핵심 시사점
- 대화 내 모든 URL을 토픽별 근거 + "🔗 공유된 URL 모음" 섹션에 설명과 함께 정리

**기능 2: 날짜별 요약 탭 기본/상세 토글 뷰**
- `[📝 기본 요약]` / `[🔍 상세 분석]` 토글 버튼으로 뷰 전환
- 기본 뷰: 기존 마크다운 요약 + `[📝 요약 생성]` 버튼
- 상세 뷰: 상세 분석 HTML 렌더링 + `[🔍 상세 생성]` `[📂 브라우저]` `[🔍 일괄 상세]` 버튼
- 브라우저 열기: 다크 테마 HTML을 시스템 기본 브라우저에서 열기

**기능 3: 다양한 상세 분석 생성 경로**
- 날짜 탭에서 단일 날짜 상세 생성 (`DetailSummaryWorker`)
- 날짜 탭에서 일괄 상세 생성 (`DetailBatchWorker`) — 기본 요약 있고 상세 없는 날짜 대상
- 도구 메뉴 → 전체 채팅방 상세 분석 생성 (`AllRoomsDetailWorker`, Ctrl+Shift+D)
- LLM 요약 생성 시 "🔍 상세 분석도 함께 생성" 체크박스 (기본 체크)

**기능 4: LLM 추론 내용 자동 제거**
- `chat_processor.py`: `<think>` 태그 제거 + `###` 이전 텍스트 제거 (기본 요약)
- `detail_prompt.py`: `<think>` 태그 제거 + `<h1>` 이전 텍스트 제거 (상세 분석)

**기능 5: 백업/복원에 상세 분석 포함**
- `file_storage.py`: 전체 백업, 채팅방별 백업, 복원 시 `detail_summary/` 디렉터리 포함

**LLM 모델 업데이트**
- GLM: `glm-4.7` → `glm-5-turbo`
- MiniMax: `MiniMax-M2.1` → `MiniMax-M2.5`

**UI 변경**
- QTextBrowser 폰트: 17px → 14px

### v2.9.0 - 기본 요약 제거, 상세 분석 전용화 (2026-04-13)

**BREAKING CHANGE: 기본 요약(마크다운) 파이프라인 완전 제거**

- 🗑️ **제거**: `llm_client.py`, `chat_processor.py`, `src/manual/` CLI 스크립트, `SummaryGeneratorWorker`, `AllRoomsSummaryWorker`, `SyncWorker`, `SummaryOptionsDialog`, `AllRoomsSummaryOptionsDialog`, `full_config.py PROMPT_TEMPLATE`
- 🗑️ **제거**: "지금 동기화" 메뉴 (Ctrl+R) — 파일 업로드와 중복
- 🗑️ **제거**: 기본/상세 토글 바 — 상세 분석 뷰가 유일한 뷰
- 🆕 **상세 분석 옵션 다이얼로그** (Ctrl+G): pending/오늘/어제~오늘/전체 범위 선택, LLM 선택, 건너뛰기 옵션
- 🔧 **`file_storage.py`**: `get_summarized_dates()`/`get_dates_needing_summary()` → `detail_summary/` 기준으로 전환
- 🔧 **`url_extractor.py`**: `extract_urls_from_html()` 추가 — HTML `<a href>` + `url-card` 구조 파싱
- 🔧 **URL 동기화**: 마크다운 파싱 → HTML 파싱으로 전환
- 🔧 **무효화 로직**: 기본 요약 참조 완전 제거, 상세 분석만 무효화
- 🔧 **전체 채팅방 상세 분석**: DB + 파일 저장소 채팅방 통합, `get_available_dates()` 기준
- 🔧 **대시보드**: 파일 I/O 제거, DB 통계만 사용하여 즉시 표시
- 📦 **보존**: `data/summary/` 기존 파일, DB `summaries` 테이블 (삭제 안 함)

---

### v2.9.1 - 상세 분석 안정성 및 타임아웃 개선 (2026-04-14)

- **API 타임아웃 상향**: 장시간 소요되는 LLM 요약이 중단되지 않도록 `full_config.py` 및 `.env.local`의 `API_TIMEOUT`을 10분(600초)에서 20분(1200초)으로 상향.
- **검증 실패 시 재시도 누락 버그 수정**: LLM 응답 검증(예: `<h2>` 태그 없음) 실패 시 재시도 로직(`continue`)을 타지 않고 즉시 실패 처리하던 버그 해결. 이제 토픽 누락 시에도 최대 3회 자동 재시도.
- **소량 대화 날짜 검증 완화**: 대화량이 극히 적은 날에도 무조건 최소 2개의 토픽을 요구하던 조건을 해제하고 최소 문자수 제한을 100자로 낮추어, 데이터 부족 시 1개 토픽만 나와도 실패 처리되지 않도록 개선.
- **마크다운 응답 자동 변환(Fallback)**: LLM이 프롬프트를 어기고 HTML `<h2>` 대신 마크다운 `##` 을 사용할 경우 에러로 버리지 않고 정규식으로 HTML 변환 후 정상 저장.
- **초장문 결과물 무반응(Silent Drop) 버그 수정**: 대화량이 너무 많아 LLM 출력 제한(`max_tokens`)에 도달하여 응답이 잘렸을 때(`finish_reason == "length"`), 기존에는 아무런 로그도 남기지 않고 몰래 실패 처리하던 크리티컬 버그 수정. 이제 잘린 응답이라도 버리지 않고 HTML 말미에 "응답 잘림 경고" UI를 삽입한 채 정상적으로 저장하여 사용자에게 투명하게 보여줌.

---

### v2.9.2 - URL 동기화 UI 개선 (2026-04-18)

- **상태바 구조 완전 재설계**: 기존 `permanent` 위젯 방식의 겹침 문제를 해결
  - 변경 전: `QStatusBar.addPermanentWidget()` 사용 → 위젯이 보이지 않거나 겹침
  - 변경 후: HBox 레이아웃 기반의 명시적 배치
    - 왼쪽: 작업 상태 ("✅ 준비", "⏳ 작업 중", "❌ 실패")
    - 중앙: 스트레치 (확장 공간)
    - 오른쪽: 마지막 동기화 시간
- **프로그레스 위젯 추가/제거 로직 개선**: 작업 시작 시 상태바 컨테이너를 임시 제거하고 프로그레스 위젯을 우선적으로 표시, 완료 후 컨테이너 복원
- **위젯 크기 명시**: `setMinimumHeight(28)`, `setMaximumHeight(28)`로 프로그레스 위젯이 명확하게 표시되도록 설정
- **두 가지 URL 동기화 모두 적용**: 
  - 채팅방별 URL 동기화 (`_sync_url_from_summaries()`)
  - 전체 채팅방 URL 동기화 (`_on_sync_all_rooms_urls()`)
- **버그 수정**: 전체 채팅방 URL 동기화에서 `all_room_names` undefined 변수 문제 해결

---

### v2.9.3 - 채팅방 선택 캐시 최적화 및 UX 개선 (2026-04-24)

- 🚀 **채팅방 데이터 캐시**: 같은 채팅방을 재클릭할 때 DB 쿼리 + 파일 I/O(날짜 탭 갱신, URL 로드 등)를 스킵하여 즉시 응답
  - `_room_cache: dict` — `{room_id: {"loaded": True}}` 구조로 로드 완료 여부 추적
  - `_on_room_selected()` 진입 시 같은 `room_id` + 캐시 유효 → 즉시 `return`
- 🔧 **캐시 무효화 전략** (`_invalidate_room_cache(room_id=None)`):
  - 상세 분석 완료 (단일/배치/전체 채팅방): 해당 `room_id` 무효화
  - 파일 업로드 완료, 채팅방 생성/삭제, DB 복구, 채팅방 복구, 백업 복원: 전체 캐시 초기화
  - 통계 갱신 (`_on_refresh_stats`): 전체 캐시 초기화
- 🎨 **채팅방 선택 하이라이트**: 선택된 채팅방에 카카오 노랑 배경 유지
  - `ChatRoomWidget.setSelected()`: Qt dynamic property `selected="true"` 설정
  - `styles.py`의 `.ChatRoomItem[selected="true"]` QSS 규칙 활용 (hover와 동일한 `#FEE500` + 왼쪽 `#3C1E1E` 보더)
  - `_highlight_selected_room(room_id)`: 사이드바 전체 위젯 순회, 선택된 방만 하이라이트
  - 캐시 히트로 `return`되더라도 하이라이트는 항상 실행
- ⚡ **체감 효과**: 대시보드 ↔ 채팅방 전환 시 불필요한 DB/파일 접근 제거로 UI 응답 즉각화

---

*마지막 업데이트: 2026-04-24 | 버전: v2.9.3*

---
> Source: [YeonHongMin/kakao-chat-summary](https://github.com/YeonHongMin/kakao-chat-summary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
