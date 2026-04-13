## openclaw-trade

> OpenClaw SQLite/SQLAlchemy DB 스키마·개발 규칙


# 데이터베이스 (DB) 규칙

> **세션 시작 전** `docs/handoff/handoff.md` 확인. 스키마 변경 시 **본 파일과 `models/*.py` Docstring**을 동시에 갱신한다.

---

## 1. 연결·스택

| 항목 | 값 |
|------|-----|
| ORM | SQLAlchemy 2.x (async) |
| 드라이버 | `aiosqlite` |
| 기본 URL | `sqlite+aiosqlite:///./openclaw.db` (`core/config.py` → `DATABASE_URL`) |
| Base | `core/database.py` → `DeclarativeBase` |
| 세션 | `get_db()` → FastAPI 의존성, commit/rollback 자동 |
| 테이블 생성 | `init_db()` → `Base.metadata.create_all` (모델 모듈 import 필수) |

**주의**: SQLite 파일 경로는 **uvicorn 실행 cwd** 기준 상대경로. 보통 `backend/`에서 실행 시 `backend/openclaw.db`. `.env`의 `DATABASE_URL`이 우선.

---

## 2. 테이블·스키마 (최신)

아래는 **코드 모델 기준** 스냅샷. 마이그레이션 파일이 없으면 기존 DB에 컬럼 추가 시 **수동 마이그레이션** 또는 DB 재생성 필요.

### 2.1 `trades` — `models/trade.py` (`Trade`)

| 컬럼 | 타입 | 설명 |
|------|------|------|
| `id` | String(36) PK | UUID |
| `symbol` | String(20), index | 예: `BTC/USDT` |
| `exchange` | String(20), index | 예: `binance` |
| `side` | String(4) | `buy` \| `sell` |
| `type` | String(10) | `market` \| `limit` |
| `amount` | Float | 수량 |
| `price` | Float | 체결/주문 가격 |
| `cost` | Float | amount × price |
| `fee` | Float, default 0 | 수수료 |
| `status` | String(10), default `open` | `open` \| `closed` \| `cancelled` \| `filled` \| `failed` (체결/실패 기록) |
| `is_paper` | Bool, default True | 페이퍼 여부 |
| `agent_id` | String(50), nullable | 실행 에이전트 |
| `strategy` | String(50), nullable | 전략명 |
| `stop_loss` | Float, nullable | 손절가 |
| `take_profit` | Float, nullable | 익절가 |
| `close_price` | Float, nullable | 청산가 |
| `pnl` | Float, nullable | 손익 |
| `signal_data` | Text, nullable | JSON 신호 |
| `created_at` | DateTime | 생성 시각 (UTC) |
| `updated_at` | DateTime | 수정 시각 (UTC) |

---

### 2.2 `portfolio_snapshots` — `models/portfolio.py` (`PortfolioSnapshot`)

| 컬럼 | 타입 | 설명 |
|------|------|------|
| `id` | Integer PK, autoincrement | |
| `total_value_usd` | Float | 총 자산 USD |
| `cash_usd` | Float | 현금 USD |
| `positions` | Text | JSON 보유 포지션 |
| `pnl_daily` | Float, default 0 | 일일 손익 |
| `pnl_total` | Float, default 0 | 누적 손익 |
| `win_rate` | Float, default 0 | 승률 % |
| `total_trades` | Integer, default 0 | 거래 수 |
| `winning_trades` | Integer, default 0 | 승리 건수 (스냅샷 시점) |
| `losing_trades` | Integer, default 0 | 패배 건수 |
| `initial_balance` | Float, default 0 | 초기 자본 USD |
| `total_return_pct` | Float, default 0 | 총 수익률 % |
| `created_at` | DateTime, index | 스냅샷 시각 (UTC) |

**참고**: `pnl_daily`는 런타임 `pnl_today`와 동일 의미로 저장됨. 기존 SQLite DB는 `init_db()` 시 컬럼 자동 추가(`ALTER`).

---

### 2.3 `agent_logs` — `models/agent_log.py` (`AgentLog`)

| 컬럼 | 타입 | 설명 |
|------|------|------|
| `id` | Integer PK, autoincrement | |
| `agent_id` | String(50), index | 에이전트 식별자 |
| `agent_type` | String(30), index | `market_analyzer` \| `strategy` \| `risk_manager` \| `execution` \| `portfolio` |
| `level` | String(10) | `INFO` \| `WARNING` \| `ERROR` \| `DECISION` \| `SIGNAL` |
| `message` | Text | 로그 메시지 |
| `data` | Text, nullable | JSON 부가 데이터 |
| `created_at` | DateTime, index | (UTC) |

---

### 2.4 `system_conditions` — `models/system_condition.py` (`SystemCondition`)

| 컬럼 | 타입 | 설명 |
|------|------|------|
| `id` | Integer PK, autoincrement | |
| `name` | String(100) | 조건식 이름 |
| `description` | Text, nullable | 설명 |
| `symbol` | String(20), default `BTC/USDT` | 대상 심볼 |
| `timeframe` | String(5), default `1h` | 타임프레임 |
| `buy_conditions` | Text, nullable | ConditionGroup JSON (매수) |
| `sell_conditions` | Text, nullable | ConditionGroup JSON (매도) |
| `is_active` | Boolean, default False | 실시간 모니터링 활성 |
| `created_at` | DateTime | |
| `updated_at` | DateTime | |
| `last_triggered_at` | DateTime, nullable | 마지막 트리거 |
| `backtest_win_rate` | Float, nullable | 캐시: 백테스트 승률 |
| `backtest_total_trades` | Integer, nullable | 캐시: 거래 수 |
| `backtest_avg_return` | Float, nullable | 캐시: 평균 수익률 |
| `backtest_max_drawdown` | Float, nullable | 캐시: MDD |
| `backtest_ran_at` | DateTime, nullable | 백테스트 실행 시각 |

**JSON 구조** (`buy_conditions` / `sell_conditions`): `logic`(`AND`/`OR`) + `conditions[]` (indicator_a, params_a, operator, type_b, value_b 등). 상세는 모델 Docstring.

---

## 3. 모델 패키지 (`models/__init__.py`)

노출 권장:

- `Trade`, `AgentLog`, `PortfolioSnapshot`, **`SystemCondition`**

`init_db()`에서 테이블 생성 시 `system_condition` 모듈이 import되어야 `system_conditions` 테이블이 생성된다.

---

## 4. 개발 규칙

### 필수
- 새 테이블: `backend/models/{이름}.py`, **snake_case** 파일명, 클래스 **PascalCase**, `__tablename__` **snake_case 복수형** 권장.
- `init_db()`에 모듈 import 추가 (미포함 시 테이블 미생성).
- 시간 컬럼: Docstring에 **UTC** 명시; 비즈니스 로직에서 KST 변환은 표시 계층에서.
- 민감정보·API키는 **DB에 저장 금지** (환경변수만).

### 권장
- Alembic 도입 시: revision 작성 후 본 `db.mdc` 스키마 섹션 동기화.
- 인덱스: 자주 필터하는 컬럼(`symbol`, `created_at`, `agent_type` 등)은 모델에 반영.

### 금지
- Raw SQL로 스키마만 어기고 모델 미갱신.
- 동기 엔진과 async 혼용 (기본은 async만).

---

## 5. ER 요약 (논리)

```
trades              — 독립 (에이전트/전략은 문자열 참조)
portfolio_snapshots — 시계열 스냅샷 (외래키 없음)
agent_logs          — 독립 로그
system_conditions   — 사용자 조건식 (백테스트 캐시 컬럼 내장)
```

현재 **테이블 간 SQLAlchemy ForeignKey 없음** (느슨한 결합).

---

*최종 스키마 동기화: 2026-03-18 — 체결 시 `trades` INSERT, `portfolio_snapshots` 확장 컬럼, `GET /api/system/conditions`에 `backtest_ran_at` 포함.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/quentinjeon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/quentinjeon)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
