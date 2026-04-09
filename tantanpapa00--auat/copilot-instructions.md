## auat

> BBooster: 트레이딩뷰 시그널 → 거래소 자동매매 플랫폼 (큐브시스템/QUBE System)

# CLAUDE.md — BBooster 프로젝트 규칙

## 프로젝트 개요
BBooster: 트레이딩뷰 시그널 → 거래소 자동매매 플랫폼 (큐브시스템/QUBE System)
- 웹 대시보드 + PC앱(Tauri/Rust) + 모바일앱
- 한국어 UI, 한국 트레이더 대상

## 절대 규칙 (NON-NEGOTIABLE)

### 1. "이미 완료됨" 스킵 금지
- 모든 작업은 반드시 grep/cat으로 현재 코드 상태를 먼저 확인한 후 시작
- 코드를 직접 확인하지 않고 "이미 되어있다"고 판단하지 마라
- 확인 결과를 반드시 출력하라

### 2. 전 거래소 일괄 적용
- 모든 코드 수정은 6개 거래소 전부 적용: OKX, Binance, Bybit, Upbit, KIS_KR, KIS_US
- 한 거래소만 수정하고 "완료"라고 하지 마라
- 커밋 전 전 거래소 동작 확인 필수

### 3. 커밋 전 필수 검증 (하나라도 실패하면 커밋 금지)
```bash
# a. 수정한 모든 .py 파일 문법 검증
python -c "import ast; ast.parse(open('파일경로').read())"

# b. 프론트엔드 빌드
cd pc-app/ui && npm run build

# c. 테스트
python -m pytest tests/ -x -q
```

### 4. VPS 정보
- IP: 76.13.180.30 (이것만 사용. 다른 IP 절대 금지)
- SSH: ssh -i ~/.ssh/bbooster_vps root@76.13.180.30
- 경로: /root/bbooster
- 배포: cd /root/bbooster && git pull && docker compose up -d --build
- 배포 후 반드시 docker logs bbooster-app --tail 30으로 확인

### 5. 문제 해결 프로세스
로그 확인 → 원인 규명 → 수정 → 재검증 → 커밋
- 원인 미파악 상태에서 다음 단계 이동 금지
- 원인 미파악 상태에서 커밋 금지
- "아마 이게 문제일 것이다"로 추측 수정 금지. 증거 기반으로만 수정

### 6. SSH/시스템 설정 변경 금지
- SSH 키, IP, 포트, nginx, Docker 설정을 임의로 변경하지 마라
- 변경이 필요하면 반드시 사용자에게 먼저 확인

### 7. 명령서 Step 검증
- 각 Step 완료 후 검증 명령 실행 결과를 반드시 보여줘라
- 검증 통과 후에만 다음 Step 이동
- 검증 실패 시 원인 파악 → 수정 → 재검증

## 프로젝트 구조

### 백엔드 (Python FastAPI)
- app/main.py — FastAPI 메인
- app/models.py — DB 모델 (CandleCache 포함)
- app/premium_routes.py — 백테스트 API
- app/strategy_engine/backtest_engine.py — 백테스트 엔진
- app/strategy_engine/candle_fetcher.py — 캔들 조회 (거래소별 API + DB 캐싱)
- app/strategy_engine/signal_generator.py — 시그널 생성
- app/strategy_engine/indicators.py — 지표 계산 (numpy 벡터화)
- app/candle_preloader.py — 자동 캐싱 (서버시작 + 1시간마다)

### 프론트엔드 (PC앱)
- pc-app/ui/index.html — PC앱 UI
- pc-app/ui/src/main.js — JS 로직
- pc-app/src-tauri/src/commands.rs — Rust HTTP client (timeout 120초)

### 웹사이트
- app/templates/home.html — qube-system.com 메인
- landing/terms.html — 이용약관
- landing/privacy.html — 개인정보처리방침

### SSOT 문서
- docs/AI_RULES.md — 개발 원칙
- docs/SSOT_HEADER.md — 일일 진행
- docs/MASTER_ROADMAP_V3.md — 전체 로드맵
- docs/FINISH_SSOT.md — 완료 공정
- docs/CONNECTOR_SPEC.md — 거래소 스펙

## 거래소별 캔들 API 제한

| 거래소 | 일봉 | 분봉 | 1회 한도 | 비고 |
|--------|------|------|---------|------|
| Binance | ✅ | ✅ (1m~1w) | 1000개 | 제한 없음 |
| OKX | ✅ | ✅ (1m~1w) | 100개 | VPS IP 429 주의 |
| Bybit | ✅ | ✅ (1m~1w) | 200개 | |
| Upbit | ✅ | ✅ (1m~1w) | 200개 | 원화마켓 |
| KIS_KR | ✅ | 당일1분봉만 | 30개 | 분봉 과거조회 불가 |
| KIS_US | ✅ | 당일1분봉만 | 30개 | 분봉 과거조회 불가 |

## 거래소별 화폐단위

| 거래소 | 화폐 | 표시 형식 | 비고 |
|--------|------|-----------|------|
| OKX | USDT | 1,234.56 USDT | 심볼 suffix로 USDC/BTC 자동 감지 |
| Binance | USDT | 1,234.56 USDT | 심볼 suffix로 USDC/BTC 자동 감지 |
| Bybit | USDT | 1,234.56 USDT | 심볼 suffix로 USDC/BTC 자동 감지 |
| Upbit | KRW | 1,234,567원 | 원화마켓 |
| KIS_KR | KRW | 1,234,567원 | 국내주식 |
| KIS_US | USD | $1,234.56 | 해외주식 |

### 화폐단위 자동 결정 로직 (getMrCurrency)
```javascript
function getMrCurrency(exchange, symbol) {
    const ex = (exchange || '').toUpperCase();
    const sym = (symbol || '').toUpperCase();

    // 거래소 기반 결정
    if (ex === 'KIS_KR' || ex === 'UPBIT') return 'KRW';
    if (ex === 'KIS_US') return 'USD';

    // 심볼 suffix 기반 결정
    if (sym.endsWith('USDC') || sym.endsWith('-USDC')) return 'USDC';
    if (sym.endsWith('USDT') || sym.endsWith('-USDT') || sym.endsWith('-SWAP')) return 'USDT';
    if (sym.endsWith('BTC') || sym.endsWith('-BTC')) return 'BTC';
    if (sym.endsWith('KRW') || sym.startsWith('KRW-')) return 'KRW';

    return 'USDT';  // 기본값
}
```

### 금액 포맷팅 (formatMrAmount)
- KRW: `1,234,567원` (소수점 없음, 천단위 콤마)
- USD: `$1,234.56` (소수점 2자리)
- USDT/USDC/BTC: `1,234.56 USDT` (소수점 2자리 + 단위)
- 만원/억 축약 사용 금지 (정확한 금액 표시)

## 빌드/배포

### 로컬 개발
- GitHub: C:\Users\pc\새 폴더\AUAT
- 빌드용: C:\AUAT\pc-app

### 빌드 순서
```
git pull
xcopy /E /Y "C:\Users\pc\새 폴더\AUAT\pc-app\ui" "C:\AUAT\pc-app\ui\"
cd C:\AUAT\pc-app\ui && npm run build
cd C:\AUAT\pc-app\src-tauri && cargo tauri build
```

### VPS 배포
```
ssh -i ~/.ssh/bbooster_vps root@76.13.180.30 "cd /root/bbooster && git pull && docker compose up -d --build"
```

## .env 필수 키
```
# 거래소 API
OKX_API_KEY, OKX_SECRET_KEY, OKX_PASSPHRASE
BINANCE_API_KEY, BINANCE_SECRET_KEY
BYBIT_API_KEY, BYBIT_SECRET_KEY
UPBIT_ACCESS_KEY, UPBIT_SECRET_KEY

# KIS (한국투자증권)
KIS_APP_KEY, KIS_APP_SECRET, KIS_ACCOUNT_NO
KIS_MOCK=true  # 모의투자 여부

# DB
DATABASE_URL=postgresql://...
```

## 캔들 DB 축적 정책
- 축적: 프리로딩 + 사용자 요청 캐싱 + 증분 갱신
- 속도: DB HIT 0.1초 vs MISS 5~15초
- 용량: 1만 종목 = 1~2GB
- 관리: 3년 이상 자동 삭제
- 테이블: candles (exchange, symbol, timeframe, timestamp, o, h, l, c, v)

## 백테스트 기준 (트레이딩뷰 전략리포트 동일)
- 상단 5카드: 총손익, 최대자본감소(MDD), 총거래수, 수익성거래%, 수익지수
- 수익률 테이블: 전체/매수/매도 3열
- 차트: Y축 수익률(%), 수익구간 초록/손실구간 빨강
- 거래 내역: 수익금 + 수익률 둘 다 표시
- 수수료 0.1% 포함
- 미실현 손익 포함 (종료 시 보유 포지션 평가)
- 화폐단위: 거래소별 자동 결정 (위 테이블 참조)

## 수익지수(Profit Factor) 표시
- >= 999 또는 Infinity → `∞` 표시
- null/undefined → `--` 표시
- 일반값 → 소수점 3자리 (예: 2.345)

## 거래소 드롭다운 한글명
| 코드 | 표시명 |
|------|--------|
| OKX | OKX |
| BINANCE | 바이낸스 |
| BYBIT | 바이비트 |
| UPBIT | 업비트 |
| KIS_KR | 한투증권(국내) |
| KIS_US | 한투증권(해외) |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tantanpapa00)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tantanpapa00)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
