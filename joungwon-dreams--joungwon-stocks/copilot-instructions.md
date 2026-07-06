## joungwon-stocks

> This file provides comprehensive context for Gemini when working on the `joungwon.stocks` project.

# GEMINI.md

This file provides comprehensive context for Gemini when working on the `joungwon.stocks` project.

## 1. Project Overview

**joungwon.stocks** is an AI-powered automated stock trading system for the Korean stock market.

- **System Name:** PROJECT AEGIS (프로젝트 이지스)
- **Philosophy:** 소중한 투자금을 리스크 관리로 보호하며 안정적인 수익 추구
- **Goal:** 데이터 수집 → AI 분석 → 자동 매매 실행
- **Status:** Phase 3 (Multi-Strategy Ensemble) 완료, 실전 운영 중
- **Key Integrations:** Korea Investment Securities (KIS) API, Google Gemini API, PostgreSQL

## 2. Current System Architecture

### 2.1 Data Collection (4-Tier System)

| Tier | Type | Technology | 주기 |
|------|------|------------|------|
| 1 | Official Libraries | pykrx, dart-fss, FinanceDataReader | 1시간 |
| 2 | Official APIs | KIS WebSocket, Naver/Daum Finance | 실시간 |
| 3 | Web Scraping | aiohttp + BeautifulSoup | 1시간 |
| 4 | Browser Automation | Playwright | 1시간 |

### 2.2 PROJECT AEGIS Components

```
src/aegis/
├── analysis/
│   ├── indicators.py      # MA, RSI, VWAP 기술 지표
│   ├── signal.py          # 점수 기반 신호 생성 (-3 ~ +3)
│   └── backtest/          # 백테스팅 엔진
├── risk/
│   ├── manager.py         # ATR 손절, Kelly Criterion, 포지션 사이징
│   └── circuit_breaker.py # 일일 손실/거래 제한
└── ensemble/
    ├── regime.py          # 시장 상태 분류 (BULL/BEAR/SIDEWAY)
    ├── registry.py        # 전략 등록/관리
    └── orchestrator.py    # 앙상블 신호 집계
```

### 2.3 Signal Scoring System

| 지표 | 조건 | 점수 |
|------|------|------|
| MA | 정배열 (20 > 60) | +1 |
| MA | 역배열 (20 < 60) | -1 |
| VWAP | 가격 > VWAP (지지) | +1 |
| VWAP | 가격 < VWAP (이탈) | -1 |
| RSI | < 30 (과매도) | +1 |
| RSI | > 70 (과매수) | -1 |

**판정:** ≥+2 강수, +1 매수, 0 관망, -1 매도, ≤-2 강도

## 3. Cron Jobs (자동화)

| Cron | 주기 | 스크립트 | 출력 |
|------|------|----------|------|
| 1분 | 08:50~15:30 (평일) | `cron/1min.py` | min_ticks + realtime_dashboard.pdf |
| 1시간 | 04:50~18:00 (매일) | `cron/1hour.py` | 종목별 PDF (10페이지) |

## 4. PDF Reports

### 4.1 realtime_dashboard.pdf (실시간 대시보드)

**생성:** `scripts/generate_realtime_dashboard_terminal_style.py`
**출력:** `reports/holding_stock/realtime_dashboard.pdf`

| 페이지 | 내용 |
|--------|------|
| 1 | 포트폴리오 요약 + 종목별 상세 (AI등급, AEGIS 신호) |
| 2 | 포트폴리오 차트 + AEGIS Market Dashboard |
| 3 | 📜 AEGIS 신호 기록 (검증 결과) |
| 4~ | 종목별 틱 데이터 |

### 4.2 {종목명}.pdf (개별 리포트)

**생성:** `scripts/gemini/generate_pdf_report.py`
**출력:** `reports/holding_stock/{종목명}.pdf`

- 10페이지 상세 분석 리포트
- 구조: `docs/PDF_STRUCTURE_SPECIFICATION.md` 참조

## 5. Database Schema

### 5.1 Core Tables

| 테이블 | 용도 |
|--------|------|
| stocks | 종목 마스터 |
| stock_assets | 보유종목 (수량, 평단가, 손익) |
| daily_ohlcv | 일봉 데이터 |
| min_ticks | 1분봉 데이터 |
| trade_history | 매매 기록 |
| aegis_signal_history | AEGIS 신호 기록 및 검증 |

### 5.2 AEGIS Signal History

```sql
aegis_signal_history (
    id, stock_code, stock_name, signal_type, score, price,
    created_at, result_1h, result_1d, is_success, verified_at
)
```

## 6. Current Holdings (2025-11-28)

| 종목 | 수량 | 평단가 | AEGIS 신호 |
|------|------|--------|------------|
| 대원전선 | 381주 | 3,634원 | 매수 (+1) |
| 한국전력 | 208주 | 46,714원 | 강수 (+2) |
| 한국카본 | 164주 | 27,792원 | 강도 (-2) |
| 롯데쇼핑 | 79주 | 71,725원 | 강수 (+2) |
| 파라다이스 | 635주 | 17,071원 | 매도 (-1) |
| 카카오 | 166주 | 57,418원 | 매도 (-1) |
| 세아홀딩스 | 12주 | 117,165원 | 부족 (0) |
| 금양그린파워 | 527주 | 11,651원 | 매수 (+1) |
| HDC현대산업개발 | 288주 | 18,582원 | 매도 (-1) |
| 우리금융지주 | 511주 | 26,083원 | 매수 (+1) |
| HD현대에너지솔루션 | 68주 | 51,071원 | 매수 (+1) |

## 7. Key Files Reference

| 파일 | 용도 |
|------|------|
| `src/aegis/` | PROJECT AEGIS 핵심 모듈 |
| `cron/1min.py` | 1분 데이터 수집 + 대시보드 |
| `cron/1hour.py` | 1시간 데이터 수집 + PDF |
| `scripts/verify_aegis_signals.py` | 신호 검증 스크립트 |
| `docs/SYSTEM_EVOLUTION_DESIGN_SPEC.md` | 시스템 설계 명세 |
| `docs/AI_COLLABORATION_LOG.md` | Claude-Gemini 협업 로그 |

## 8. Role Division

- **Gemini:** 설계자 (Architect) - 고도화 로드맵, 전략 설계
- **Claude:** 구현자 (Implementer) - MVP 코드 구현

## 9. Pending Tasks

1. Phase 3.5: `WeightOptimizer`, `RobustnessTester` 구현 확인
2. AEGIS 신호 검증 (`scripts/verify_aegis_signals.py`) 정기 실행
3. 첫 AEGIS 실전 매수 (롯데쇼핑 5주) 성과 모니터링

---

*Last Updated: 2025-11-28*

---
> Source: [joungwon-dreams/joungwon-stocks](https://github.com/joungwon-dreams/joungwon-stocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-05 -->
