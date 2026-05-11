## poskds

> 주방 KDS(Kitchen Display System) 조리중 건수를 Firebase에 업로드. 주방폰에서 실행.

# PosKDS 프로젝트 규칙

## 앱 개요
주방 KDS(Kitchen Display System) 조리중 건수를 Firebase에 업로드. 주방폰에서 실행.

## Firebase 구조
- **DB**: `poskds-4ba60-default-rtdb.asia-southeast1.firebasedatabase.app`
- **건수 업로드**: `/kds_status.json` — count, completed, time, orders 배열, source
- **FCM Push**: 건수 변동 시 `FcmSender.kt`가 FCM v1 API로 PosDelay에 push (토픽 `kds_push`)
- **로그**: `/kds_log.json`
- **원격 업데이트**: `/app_update/poskds.json` — version, url

## 로그 확인 (원격)
```bash
# 로그
curl -s https://poskds-4ba60-default-rtdb.asia-southeast1.firebasedatabase.app/kds_log.json
# 건수
curl -s https://poskds-4ba60-default-rtdb.asia-southeast1.firebasedatabase.app/kds_status.json
```

## 웹 대시보드
- GitHub Pages: `https://wk7007-wk.github.io/PosKDS/`
- 파일: `/root/PosKDS/docs/index.html`
- PosDelay 앱의 WebView가 이 대시보드를 로드
- 조리모드, 모니터링, 광고 제어 UI 포함

## 주방폰 제약
- **물리적으로 원격** — 직접 접근 어려움 (매장)
- 로그는 Firebase로만 접근 가능
- **KDS 앱 수정/업데이트 최소화** — 데이터 보정은 항상 수신 측(PosDelay/웹)에서 처리
- 웹 대시보드 수정 시 앱 재설치 불필요 (GitHub Pages 자동 반영)

## KDS 원격 업데이트 (APK 배포)
KDS 앱이 Firebase `/app_update/poskds.json`을 감지하여 자동 다운로드 + 설치 화면 표시.
```bash
# 1. GitHub Release 업로드
VER=$(date +%y%m%d.%H%M)
gh release create "v${VER}" /sdcard/Download/PosKDS.apk --title "v${VER} 설명" --repo wk7007-wk/PosKDS

# 2. Firebase에 업데이트 URL 등록 → 주방폰 자동 감지
NOW=$(date +"%Y-%m-%d %H:%M:%S")
URL="https://github.com/wk7007-wk/PosKDS/releases/download/v${VER}/PosKDS.apk"
curl -s -X PUT "https://poskds-4ba60-default-rtdb.asia-southeast1.firebasedatabase.app/app_update/poskds.json" \
  -d "{\"version\":\"${VER}\",\"url\":\"${URL}\",\"time\":\"${NOW}\"}"
```
**반드시 2단계 모두 수행** — Release만 올리면 주방폰이 감지 못함.

## KDS 데이터 신뢰도
- **탭 건수(조리중 N) 신뢰**: KDS `count` 값은 탭 헤더에서 추출 — 가장 정확한 소스
- **orders 배열 불안정**: `rootInActiveWindow`가 systemui 반환 시 orders=[] (주문번호 추출 실패)
- **count>0 + orders=[] → 0 보정 금지**: 탭 건수가 정확, orders 빈배열은 추출 실패일 뿐
- **추출 실패 시 이전 값 유지**: count=null + orders=[] → UI 전환 중 일시적 실패. 절대 0으로 강제하지 않음
- `orders` 없이 `count`만 올 때 → 30분간 건수 변동 없으면 0으로 강제 보정 (웹 대시보드)
- `orders` 있을 때 → 25분 초과 개별 주문 차감 (기존 필터)
- **KDS 건수 즉시 반영**: 안정화 지연 삭제됨 — PosDelay/웹 모두 즉시 적용
- **completed 필드**: 완료탭 건수 추출 (트리탐색 fallback), 조리완료 버튼 누를 때 증가

## PackHelper 연동 경계 (0409)
- 공유 인터페이스: `/kds_status.json` (KDS→쓰기, PackHelper→읽기)
- 상세: `rules/packhelper_interface.txt` (수정범위, 금지영역, 동기화 규칙)

## KDS rootInActiveWindow 문제 (핵심 버그 패턴)
- **증상**: KDS가 조리중 건수를 감지했다가 수초 내 0으로 되돌림
- **원인**: `rootInActiveWindow`가 KDS 앱이 아닌 systemui/잠금화면 반환 → 추출 실패
- **해결 (KDS 측)**: `findKdsRoot()` — 패키지명으로 윈도우 탐색 (`windows` API + `flagRetrieveInteractiveWindows`)
- **해결 (수신 측)**: count>0+orders=[] 신뢰, 0 안정화 90초, 추출 실패 시 이전 값 유지
- **교훈**: 접근성 서비스에서 `rootInActiveWindow`는 항상 대상 앱 윈도우를 반환하지 않음. 교차검증으로 0을 강제하면 오히려 정확한 값을 덮어씀

---
> Source: [wk7007-wk/PosKDS](https://github.com/wk7007-wk/PosKDS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
