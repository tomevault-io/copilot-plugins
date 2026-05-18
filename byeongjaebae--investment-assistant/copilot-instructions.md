## investment-assistant

> 이 파일은 다음 에이전트가 프로젝트 지식을 빠르게 찾기 위한 인덱스다. 자세한 내용은 링크된 문서를 읽는다.

# AGENTS.md

이 파일은 다음 에이전트가 프로젝트 지식을 빠르게 찾기 위한 인덱스다. 자세한 내용은 링크된 문서를 읽는다.

## Start Here

- Session memory: `memory/session.md`
- Main decision record: `docs/decisions_and_evidence.md`
- Current strategy: `docs/dip_accumulation_strategy.md`
- Agent design: `docs/agent_design.md`

## Project Map

| Topic | Read |
| --- | --- |
| 전체 결정사항과 근거 | `docs/decisions_and_evidence.md` |
| 장기보유형 조정 매수 전략 | `docs/dip_accumulation_strategy.md` |
| 에이전트 역할 분리 | `docs/agent_design.md` |
| PWA 기반 알림 아키텍처 | `docs/pwa_notification_architecture.md` |
| 정책 리서치 계획과 기존 기준선 | `docs/policy_research_plan.md` |
| 외부 참고자료 | `docs/research_sources.md` |
| finance 전문가 리뷰 패킷 | `reports/finance_expert_analysis_pack.md` |
| 세션 인수인계 규칙 | `skills/session-handoff/SKILL.md` |

## Code Index

| Component | Path | Purpose |
| --- | --- | --- |
| CLI | `src/invest_assistant/cli.py` | `fetch`, `backtest`, `dip-backtest`, `scan` 명령 |
| Data loading | `src/invest_assistant/data.py` | Yahoo Finance 데이터 수집/캐시 |
| Indicators | `src/invest_assistant/indicators.py` | SMA, RSI, MACD, ATR, drawdown 재료 |
| Old baseline backtest | `src/invest_assistant/backtest.py` | 기존 매매형 기준선 |
| Dip accumulation backtest | `src/invest_assistant/dip_backtest.py` | 적립식/조정매수/레버리지 전술 백테스트 |
| Timing agent | `src/invest_assistant/agents/timing_advisor.py` | 조정 매수 조건 점수화 |
| Long-term finder | `src/invest_assistant/agents/long_term_finder.py` | 장기 후보 점수화 |
| Thesis reviewer | `src/invest_assistant/agents/thesis_reviewer.py` | thesis 훼손 여부 판단 |

## Config Index

| Config | Purpose |
| --- | --- |
| `configs/agents.yaml` | 에이전트 역할, 데이터 소스, 정책 변형 |
| `configs/strategy.yaml` | 기존 기준선 전략 설정 메모 |
| `configs/watchlist.txt` | 기본 관심 종목 |

## Report Index

| Report | Purpose |
| --- | --- |
| `reports/dip_backtest_summary.csv` | 장기보유형 조정매수 백테스트 요약 |
| `reports/dip_backtest_report.md` | 장기보유형 조정매수 백테스트 설명 |
| `reports/dip_backtest_trades.csv` | 장기보유형 조정매수 매수 내역 |
| `reports/dip_backtest_monthly.csv` | 월별 포트폴리오 스냅샷 |
| `reports/portfolio_backtest_report.md` | 근거 기반 포트폴리오 백테스트 설명 |
| `reports/finance_expert_analysis_pack.md` | finance 전문가 에이전트용 분석 패킷 |
| `reports/backtest_summary.csv` | 기존 매매형 기준선 요약 |
| `reports/trades.csv` | 기존 매매형 거래 내역 |

## Canonical Commands

```bash
.venv/bin/python -m pytest
.venv/bin/python main.py dip-backtest --start 2020-01-01 --end 2026-05-01
.venv/bin/python main.py backtest --tickers AAPL MSFT NVDA --start 2020-01-01 --end 2025-12-31
```

## Strategy Rules

- 기술지표는 매도 신호가 아니라 조정 매수 판단용이다.
- 개별주 매도는 `ThesisReviewAgent` 또는 Research Agent가 뉴스/실적/어닝콜/재무제표/가이던스로 thesis 훼손을 확인했을 때만 검토한다.
- 지수 ETF는 원칙적으로 적립식 매수와 조정 추가매수 중심이다.
- `QLD`, `TQQQ`는 장기 핵심 보유가 아니라 QQQ 조정 구간의 전술 자산이다.
- 모든 결과는 투자 조언이 아니라 리서치와 가설 검증 자료로 표현한다.

## Handoff

Before ending substantial work, update `memory/session.md`:

```bash
python3 skills/session-handoff/scripts/handoff.py end --note "short note"
```

---
> Source: [ByeongjaeBae/investment-assistant](https://github.com/ByeongjaeBae/investment-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
