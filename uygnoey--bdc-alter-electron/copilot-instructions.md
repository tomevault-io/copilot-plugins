## bdc-alter-electron

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Language Preferences / 언어 설정

- **Responses**: Always respond in Korean (한국어로만 답변)
- **Documentation**: Write all documentation, comments, and code documentation in both English and Korean (문서는 영어와 한국어 모두 작성)

## Project Overview

BMW 드라이빙 센터 예약 모니터링 시스템 - Electron + React + Tailwind CSS v4 기반 데스크톱 애플리케이션
BMW Driving Center reservation monitoring system - Desktop application based on Electron + React + Tailwind CSS v4

## Commands

### Package Management / 패키지 관리
- **IMPORTANT**: Use `bun` for all package management (not npm or yarn)
- **중요**: 모든 패키지 관리는 `bun`을 사용합니다 (npm이나 yarn 대신)

### Build
```bash
bun run build
```
Compiles TypeScript files from `src/` to JavaScript in `dist/` directory.

### Run Electron App / 일렉트론 앱 실행
```bash
bun run start
# or / 또는
bun run dev
```
Starts the Electron application / 일렉트론 애플리케이션을 시작합니다

## Project Structure

- `electron/` directory:
  - `main/index.js`: Electron main process with BMW automation (BMW 자동화가 포함된 메인 프로세스)
  - `main/bmw-programs-parser.js`: BMW 프로그램 목록 파싱 모듈
  - `main/solvecaptcha-wrapper.js`: SolveCaptcha API wrapper
  - `preload/index.js`: IPC bridge for renderer process (렌더러 프로세스용 IPC 브리지)
- `src/` directory:
  - `components/ElectronBrowser.tsx`: Main browser component with split view
  - `components/BMWReservationPanel.tsx`: BMW reservation settings panel
  - `types/electron.d.ts`: TypeScript type definitions
- Vite + React + TypeScript configuration
- Tailwind CSS v4 with PostCSS

## Development Progress Tracking / 개발 진행 상황 추적

### Completed Tasks / 완료된 작업
1. ✅ Electron + React + Tailwind CSS v4 기반 데스크톱 앱 구현
2. ✅ 50:50 분할 화면 레이아웃 (BMW 예약 설정 패널 / BrowserView)
3. ✅ BMW OAuth 자동 로그인 (이메일 → 비밀번호 → hCaptcha)
4. ✅ SolveCaptcha API 통합 (hCaptcha 자동 해결)
5. ✅ 프로그램 목록 파싱 (Experience, Training, Owner 카테고리 - Junior Campus 제외)
6. ✅ 다중 프로그램 선택 UI (체크박스 형태)
7. ✅ 예약 모니터링 기능
8. ✅ 사용하지 않는 코드 정리 및 최적화

## Key Features / 주요 기능

### BMW 예약 모니터링 / BMW Reservation Monitoring
- BMW OAuth 자동 로그인 (2단계: 이메일 → 비밀번호)
- hCaptcha 자동 감지 및 SolveCaptcha API 연동
- 프로그램 목록 실시간 파싱 (테이블 헤더 기반)
- 다중 프로그램 선택 가능 (체크박스 UI)
- 예약 가능 여부 주기적 확인 (30초/1분/2분/5분)
- 알림 기능 (브라우저 노티피케이션)

### 기술적 특징 / Technical Features
- Electron BrowserView (iframe 대신)
- IPC 통신으로 메인/렌더러 프로세스 분리
- TypeScript + React 19.1.1
- Tailwind CSS v4 + PostCSS
- Vite 번들링

## Important Notes / 중요 사항

### BMW 사이트 구조
- 로그인 URL: https://oneid.bmw.co.kr
- 프로그램 페이지: https://driving-center.bmw.co.kr/useAmount/view
- 프로그램 카테고리: Experience, Training, Owner (Junior Campus는 파싱 제외)
- rowspan 처리 필요 (예: Taxi 프로그램)

### 환경 변수 / Environment Variables
`.env` 파일에 BMW 계정 정보 저장:
```
BMW_USERNAME=your_email@example.com
BMW_PASSWORD=your_password
SOLVECAPTCHA_API_KEY=your_api_key
```

### 주의사항
- hCaptcha는 로그인 버튼 클릭 **후** 나타남
- Vue.js SPA이므로 페이지 로드 대기 필요
- 프로그램명 파싱 시 "분", "원" 등 제외 필요

## GitHub Repository
- URL: https://github.com/uygnoey/bdc-alter-electron
- Main branch: main

## Latest Updates (2025-08-22) / 최신 업데이트

### 1. Multi-Category Program Parsing / 모든 카테고리 프로그램 파싱
- **All Categories Iteration**: Now checks Experience, Training, Owner, Test Drive, Off-Road
- **모든 카테고리 순회**: Experience, Training, Owner, Test Drive, Off-Road 모두 확인
- Each category is clicked and activated before parsing
- 각 카테고리를 클릭하여 활성화 후 파싱

### 2. Continuous Monitoring / 연속 모니터링
- **No Intervals**: Immediately starts next check after completion
- **주기 없음**: 완료 즉시 다음 확인 시작
- Uses `useRef` to maintain monitoring state
- Force stop with AbortController implementation

### 3. Program Filtering / 프로그램 필터링
- **Exact Name Matching**: "Starter Pack" ≠ "i Starter Pack"
- **정확한 이름 매칭**: "Starter Pack" ≠ "i Starter Pack"
- Language variants supported: (KOR), (ENG)
- Only selected programs are shown in results

### 4. Date Format / 날짜 형식
- **Full Date Display**: "2025년 08월 22일" format
- **전체 날짜 표시**: "2025년 08월 22일" 형식
- monthYear info passed from backend to frontend

### 5. Build System / 빌드 시스템
- **Electron Builder**: Multi-platform support (macOS, Windows, Linux)
- **Auto-updater**: GitHub Releases integration
- **Icons**: Generated from SVG → PNG for all platforms
- **Config**: electron-builder.yml configuration file

### Build Commands / 빌드 명령어
```bash
bun run dist:mac     # macOS build
bun run dist:win     # Windows build  
bun run dist:linux   # Linux build
bun run release      # GitHub release
```

### Version Info
- Current: 0.0.1
- App ID: me.yeongyu.bmw.driving.center.monitor
- Auto-update: Checks on startup and every 30 minutes

## Recent Major Refactoring (2025-08-21) / 이전 주요 리팩토링

### Program Data Structure / 프로그램 데이터 구조
- **Before**: `{ id: string, name: string, category: string }[]`
- **After**: `string[]` (simple array of program names)
- **변경**: 객체 배열에서 단순 문자열 배열로

### Current Flow / 현재 플로우
1. User clicks "프로그램 가져오기" → Fetches programs from useAmount/view
2. User selects programs and starts monitoring
3. System navigates to schedule page → Auto-login if needed → Check reservations
4. All categories are checked for selected programs
5. Results show only selected programs with full date format

## Important Implementation Details / 중요 구현 세부사항

### Program Name Matching Logic / 프로그램명 매칭 로직
```javascript
// Exact matching with language variants
"Starter Pack" matches: "Starter Pack (KOR)", "Starter Pack (ENG)"
"Starter Pack" does NOT match: "i Starter Pack", "Starter Pack Plus"

// M series programs
"M Town" matches: "M Town (KOR)", "M Town (ENG)" 
"M Town" does NOT match: "M Town Driving"
```

### Continuous Monitoring Implementation / 연속 모니터링 구현
```javascript
const isRunningRef = useRef(false);
const continuousMonitoring = async () => {
  while (isRunningRef.current) {
    await checkReservation();
    // No delay, immediately continue
  }
}
```

### Force Stop with AbortController / 강제 중단
```javascript
let monitoringAbortController = new AbortController();
// Check throughout execution:
if (monitoringAbortController.signal.aborted) {
  throw new Error('중단됨');
}
```

## Known Issues & Considerations / 알려진 이슈 및 고려사항

### Current Limitations / 현재 제한사항
- hCaptcha requires manual solving (SolveCaptcha API integration incomplete)
- Windows and Linux builds need testing
- Code signing for macOS distribution pending

## Key Files and Their Roles / 주요 파일과 역할

### Core Files
- `electron/main/browser-automation.js`: BMW site automation logic / BMW 사이트 자동화 로직
- `src/components/BMWReservationPanel.tsx`: UI for reservation monitoring / 예약 모니터링 UI
- `electron-builder.yml`: Build configuration / 빌드 설정
- `build/`: Icons and resources / 아이콘 및 리소스
- `PROJECT_MEMORY.md`: Detailed project documentation in English / 영문 상세 프로젝트 문서

## Critical Notes / 중요 참고사항
- **NEVER** use npm or yarn - always use `bun`
- Program names are now simple string arrays, not objects
- Login only happens when redirected (not proactively)
- All responses should be in Korean (한국어)
- Don't create documentation unless explicitly requested

## TODO - Next Implementation / 다음 구현 사항

### 1. Enhanced Notifications / 알림 개선
- Email notifications
- Telegram/Discord integration
- Sound alerts

### 2. Auto-reservation / 자동 예약
- Automatic seat booking when available
- Captcha solving automation improvement
- Payment integration

---
> Source: [uygnoey/bdc-alter-electron](https://github.com/uygnoey/bdc-alter-electron) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
