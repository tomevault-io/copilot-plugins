## quant-agent

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

자율 주식 매매 에이전트 시스템. 처음부터 AI에게 매매를 맡기지 않고, **결정론적 데이터 파이프라인(Phase 0~1)을 먼저 완성**한 뒤 그 위에 LLM 에이전트(Phase 2+)를 단계적으로 얹는다. 현재 상태: **Phase 0 스캐폴딩 시작 단계**.

## 개발 명령어

패키지 관리는 **uv** 사용. 프로젝트가 스캐폴딩되면 아래 명령어를 사용한다.

```bash
uv sync                        # 의존성 설치
uv run pytest                  # 전체 테스트
uv run pytest tests/unit/test_price.py  # 단일 테스트 파일
uv run pytest -k "test_fetch"  # 특정 테스트만
uv run python -m scheduler     # 스케줄러 실행
uv run ruff check .            # 린트
uv run ruff format .           # 포맷
```

시크릿은 `.env`에서 로드. 시작 시 필수 키 누락이면 즉시 실패한다.

## 아키텍처 핵심

### AI 경계

```
Phase 2+: LLM 에이전트 (관찰·판단·제안)
══════════════════════════════════════  ← AI 경계
Phase 0~1: 결정론적 파이프라인 (AI 없음)
```

**AI 경계 아래는 같은 입력 → 같은 출력이 보장**돼야 한다. LLM이 무엇을 보고 판단했는지 재현·감사할 수 있어야 하기 때문이다. Phase 0~1에 LLM을 추가하지 않는다.

### 세 개의 핵심 인터페이스

모든 교체 가능성은 이 세 인터페이스가 책임진다.

**`Collector`** (`collectors/base.py`) — 데이터 소스 추상화
- `fetch(symbol, start, end) -> DataFrame` — 불변 반환
- `supports(market) -> bool` — KR / US
- 첫 구현체: `FinanceDataReader` (KR+US 단일 API). 나중에 KIS/Alpaca 추가해도 인터페이스 불변.

**`Store`** (`storage/base.py`) — 저장소 추상화
- `upsert(table, df)` — 멱등 저장 (같은 데이터 두 번 넣어도 중복 없음)
- `query(sql) -> DataFrame`
- 첫 구현체: DuckDB (파일 1개). 규모 커지면 TimescaleDB로 교체해도 상위 레이어 무관.

**`AlertChannel`** (`alerts/base.py`) — 알림 채널 추상화
- `send(message)`, `send_report(report)`
- Phase 4에서 `request_approval(proposal) -> decision` 추가 (Discord 버튼 게이트)
- 첫 구현체: Discord webhook. Discord를 선택한 이유는 멀티에이전트 페르소나·스레드·승인 버튼이 Telegram보다 자연스럽기 때문.

### 데이터 흐름 (Phase 0~1)

```
scheduler → universe.tiers → collectors → storage (DuckDB)
                                                  ↓
                                             analysis (지표)
                                                  ↓
                                    alerts.rules     reports
                                          ↓              ↓
                                       AlertChannel (Discord)
```

**LLM은 채널을 읽지 않는다.** Phase 2+에서 LLM은 `storage`에서 직접 구조화된 데이터를 읽는다. Discord는 사람을 위한 출력 채널이다.

### 유니버스 구조

종목 리스트가 아니라 **선정 기준을 config로** 관리한다.
- **Tier 1**: 수동 큐레이션 ~25종목 (항상 추적)
- **Tier 2**: 조건 기반 동적 스캔 (거래대금·변동성 기준 로테이션)

스윙/단기(최대 2~3개월) 중심인 이유: 수익 최적이 아니라 **학습 속도 최적** — 완결된 매매 사이클이 빨리 쌓여야 전략·에이전트 개선 루프가 돌아간다.

## 불변 원칙

- **수집은 종목 단위로 격리** — 한 종목 실패가 전체 파이프라인을 죽이지 않는다.
- **잡은 멱등** — 같은 잡을 두 번 돌려도 중복 저장 없음.
- **분석 함수는 순수** — `analysis/` 아래 모든 함수는 입력을 변경하지 않고 새 객체를 반환한다.
- **시크릿은 `.env`만** — 코드에 하드코딩 금지, 시작 시 존재 검증.

## Phase별 완료 기준

각 Phase는 DoD(Definition of Done)를 충족해야 다음으로 넘어간다. 자세한 내용은 [docs/roadmap.md](docs/roadmap.md).

| Phase | 핵심 DoD |
|-------|---------|
| 0 | Tier 1 OHLCV가 매일 자동 적재, 멱등, 종목 단위 장애 격리 |
| 1 | 규칙 조건 충족 시 Discord 알림, 정기 리포트 자동 생성 |
| 2 | LLM이 근거 있는 제안 생성, 모의 장부 추적, 입력 컨텍스트 재현 가능 |
| 3 | 역할 에이전트 토론·합의, Discord 스레드 추적 |
| 4 | 모든 주문에 승인 게이트, 리스크 한도 코드 강제, 킬 스위치 즉시 동작 |

## 설치된 ECC 컴포넌트

프로젝트 레벨 `.claude/`에 Python 퀀트 프로젝트 전용으로 설치됨.

**주요 에이전트**: `python-reviewer`, `silent-failure-hunter` (파이프라인 조용한 실패 탐지), `database-reviewer` (DuckDB 쿼리), `security-reviewer`, `tdd-guide`, `performance-optimizer`

**주요 스킬**: `llm-trading-agent-security` (트레이딩 에이전트 보안), `data-scraper-agent`, `tdd-workflow`, `verification-loop`, `deployment-patterns`

---
> Source: [HaeDalWang/Quant-Agent](https://github.com/HaeDalWang/Quant-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
