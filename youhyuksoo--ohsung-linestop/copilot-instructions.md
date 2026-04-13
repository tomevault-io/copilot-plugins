## ohsung-linestop

> "오성 라인 정지 시스템"은 스마트 팩토리 환경에서 생산 라인의 불량 발생을 실시간으로 모니터링하고, 특정 조건 충족 시 자동으로 생산 라인을 정지시키는 웹 기반 애플리케이션입니다. 이 시스템은 제조 공정의 품질 관리 및 안정성 확보를 목표로 합니다.

# 오성 라인 정지 시스템 개발 문서

## 1. 프로젝트 개요

"오성 라인 정지 시스템"은 스마트 팩토리 환경에서 생산 라인의 불량 발생을 실시간으로 모니터링하고, 특정 조건 충족 시 자동으로 생산 라인을 정지시키는 웹 기반 애플리케이션입니다. 이 시스템은 제조 공정의 품질 관리 및 안정성 확보를 목표로 합니다.

### 1.1. 주요 기능

*   **실시간 라인 모니터링:** 생산 라인의 현재 상태(가동/정지) 및 실시간 불량 발생 현황을 대시보드를 통해 제공합니다.
*   **불량 데이터 수집 및 분석:** Oracle DB로부터 불량 데이터를 주기적으로 수집하고, 정해진 규칙에 따라 불량 유형별 카운트를 집계합니다.
*   **자동 라인 정지:** 불량 코드 및 임계값을 기반으로 정의된 규칙에 따라 임계값을 초과하는 불량이 감지될 경우, 미쓰비시 PLC로 정지 신호를 전송하여 라인을 자동으로 멈춥니다.
*   **불량 이력 관리:** 발생했던 모든 불량 이력을 조회하고, 기간 및 불량 코드별로 필터링하여 과거 데이터를 분석할 수 있습니다.
*   **관리자 기능:** 모니터링 서비스 시작/정지, 불량 규칙(Defect Rule)의 생성/수정/삭제, PLC 강제 제어(정지/재가동) 등의 관리자 기능을 제공합니다.
*   **사용자 인증 및 권한 관리:** 안전한 시스템 운영을 위해 사용자 로그인 및 관리자 권한을 통해 특정 기능에 대한 접근을 제한합니다.
*   **시스템 설정 관리:** PLC 연결 정보, DB 연결 정보, 폴링 주기 등 시스템 운영에 필요한 각종 설정을 웹 UI를 통해 관리할 수 있습니다.
*   **알림 기능:** 라인 정지 등 중요 이벤트 발생 시 사용자에게 실시간 알림을 제공합니다.
*   **Mock 모드 지원:** 실제 PLC 및 DB 연결 없이도 개발 및 테스트를 수행할 수 있도록 Mock 모드를 지원합니다.

### 1.2. 기술 스택

*   **프론트엔드 프레임워크:** Next.js (React 기반)
*   **언어:** TypeScript
*   **스타일링:** Tailwind CSS
*   **상태 관리:** Zustand
*   **백엔드 (API Routes):** Next.js API Routes (Node.js 환경)
*   **데이터베이스:** Oracle DB
*   **PLC 통신:** Mitsubishi MC Protocol (3E/4E Frame) (Node.js `mcprotocol` 라이브러리)
*   **HTTP 클라이언트:** Axios
*   **날짜 처리:** `date-fns`
*   **로그 관리:** 자체 `logger` 서비스
*   **타입스크립트 유틸리티:** `clsx`, `tailwind-merge`

### 1.3. 시스템 아키텍처 개요

본 시스템은 Next.js의 Full-stack 기능을 활용하여 프론트엔드와 백엔드 API를 통합 구현했습니다.

1.  **클라이언트 (브라우저):** 사용자 인터페이스를 렌더링하고 API를 통해 백엔드와 통신합니다.
2.  **Next.js 서버 (API Routes):**
    *   인증, 관리자 제어, 불량 이력 조회 등의 API 엔드포인트를 제공합니다.
    *   `src/lib/services`에 정의된 핵심 비즈니스 로직(모니터링, DB, PLC 통신)을 호출합니다.
3.  **모니터링 서비스 (`src/lib/services/monitor.ts`):**
    *   시스템의 핵심 로직을 담당하는 싱글톤 서비스입니다.
    *   설정된 폴링 주기(`settings.json` 또는 기본 30초)에 따라 동작합니다.
    *   주기적으로 Oracle DB 또는 Mock DB에서 불량 데이터를 조회합니다.
    *   조회된 불량 데이터와 미리 정의된 불량 규칙을 비교하여 라인 정지 여부를 결정합니다.
    *   라인 정지 조건 충족 시 PLC 서비스(`src/lib/services/plc.ts`)를 통해 실제 PLC에 정지 명령을 전송합니다.
    *   시스템의 현재 상태(가동 여부, 라인 상태, 불량 카운트 등)를 관리하고 API를 통해 제공합니다.
4.  **DB 서비스 (`src/lib/services/db.ts`):**
    *   Oracle DB와의 연결 및 데이터(불량, 규칙) 조회/저장을 담당합니다.
    *   `ICOM_RECIEVE_DATA_NG` 테이블에서 불량 데이터를 조회하고, `SP_CHECK_LINE_STOP` 프로시저를 호출하여 라인 정지 판단 로직을 수행합니다.
    *   `defect_rules.json` 파일에 불량 규칙을 영구 저장하고 관리합니다.
    *   테스트를 위한 Mock DB 기능을 포함합니다.
5.  **PLC 서비스 (`src/lib/services/plc.ts`):**
    *   미쓰비시 PLC와의 실제 통신(MC Protocol)을 담당합니다.
    *   PLC의 특정 비트 주소(`settings.json` 또는 기본 `M100`)에 값을 쓰거나 읽어서 라인 상태를 제어(정지/재가동)하고 확인합니다.
    *   테스트를 위한 Mock PLC 기능을 포함합니다.
6.  **세션 및 알림 스토어:** Zustand를 사용하여 클라이언트와 서버 간의 세션 정보 및 전역 알림 상태를 관리합니다.

이러한 구성은 모듈성과 확장성을 고려하여 설계되었으며, 각 서비스는 독립적인 책임을 가집니다.

## 2. 폴더 구조 및 역할

프로젝트는 Next.js 표준 폴더 구조를 따르며, 주요 기능 및 책임에 따라 폴더가 구성되어 있습니다.

```
D:\\Project\\OhSung-LineStop\\
├───.next\\                # Next.js 빌드 및 캐시 파일
├───node_modules\\          # 프로젝트 의존성 모듈
├───public\\                # 정적 파일 (이미지, 폰트 등)
├───src\\                   # 모든 소스 코드
│   ├───app\\               # Next.js App Router (페이지 및 API 라우트)
│   │   ├───api\\           # API 라우트 정의 (백엔드 역할)
│   │   │   ├───admin\\     # 관리자 관련 API (서비스 제어, 규칙 관리 등)
│   │   │   │   ├───control\\   # 모니터링 서비스 시작/정지 등 제어
│   │   │   │   ├───rules\\     # 불량 규칙 CRUD
│   │   │   │   └───[code]\\    # 특정 불량 코드 규칙 처리
│   │   │   ├───auth\\      # 인증 관련 API (로그인, 로그아웃, 사용자 정보)
│   │   │   │   ├───login\\
│   │   │   │   ├───logout\\
│   │   │   │   └───me\\
│   │   │   ├───db-test\\   # 데이터베이스 테스트용 API
│   │   │   ├───defects\\   # 불량 이력 조회 API
│   │   │   ├───init\\      # 초기화 관련 API
│   │   │   ├───logs\\      # 로그 조회 API
│   │   │   ├───notifications\\ # 알림 관련 API
│   │   │   │   └───stream\\ # SSE (Server-Sent Events) 스트림
│   │   │   ├───plc-test\\  # PLC 테스트용 API
│   │   │   ├───resolve\\   # 불량 해결 API
│   │   │   ├───settings\\  # 시스템 설정 관리 API
│   │   │   └───status\\    # 시스템 상태 조회 API
│   │   ├───(pages)\\       # 라우팅 가능한 UI 페이지 (실제 폴더명은 pages가 아님)
│   │   │   ├───admin\\     # 관리자 페이지
│   │   │   ├───defects\\   # 불량 이력 페이지
│   │   │   ├───help\\      # 도움말 페이지
│   │   │   ├───login\\     # 로그인 페이지
│   │   │   ├───logs\\      # 로그 페이지
│   │   │   ├───monitor\\   # 실시간 모니터링 대시보드 페이지
│   │   │   └───settings\\  # 시스템 설정 페이지
│   │   ├───globals.css\\   # 전역 CSS 스타일
│   │   ├───layout.tsx\\    # 모든 페이지에 적용되는 루트 레이아웃
│   │   └───page.tsx\\      # 메인 랜딩 페이지
│   ├───components\\        # 재사용 가능한 UI 컴포넌트
│   │   ├───monitor\\       # 모니터링 페이지 전용 컴포넌트
│   │   ├───BackButton.tsx
│   │   ├───LoginForm.tsx
│   │   ├───Navbar.tsx      # 상단 네비게이션 바
│   │   ├───NotificationDropdown.tsx
│   │   ├───ProtectedRoute.tsx # 로그인 보호 라우트
│   │   ├───ThemeToggle.tsx # 다크/라이트 모드 토글
│   │   └───UserMenu.tsx    # 사용자 메뉴 드롭다운
│   └───lib\\               # 비즈니스 로직 및 유틸리티
│       ├───services\\      # 핵심 서비스 로직 (DB, PLC, 모니터링 등)
│       │   ├───db.ts\\         # 데이터베이스 인터페이스 및 규칙 관리
│       │   ├───logger.ts\\     # 로깅 유틸리티
│       │   ├───monitor.ts\\    # 핵심 모니터링 서비스 (라인 정지 로직)
│       │   ├───plc.ts\\        # PLC 통신 인터페이스
│       │   └───state.ts\\      # (현재는 사용되지 않거나 단순화된 상태 관리)
│       ├───store\\           # Zustand 기반 전역 상태 관리
│       │   ├───monitor.ts\\    # 모니터링 상태 스토어
│       │   ├───notification-store.ts\\ # 알림 상태 스토어
│       │   └───session-store.ts\\   # 세션 상태 스토어
│       ├───auth.ts\\         # 인증 관련 유틸리티 함수
│       ├───settings.ts\\     # 시스템 설정 유틸리티
│       ├───types.ts\\        # 타입 정의 (인터페이스, Enum 등)
│       └───utils.ts\\        # 공통 유틸리티 함수
├───defect_rules.json       # 불량 규칙 데이터 (DB 서비스에 의해 관리)
├───next.config.js          # Next.js 설정 파일
├───package.json            # 프로젝트 의존성 및 스크립트
├───postcss.config.js       # PostCSS 설정 (Tailwind CSS 포함)
├───settings.json           # 시스템 전역 설정 (DB, PLC, Mock 모드 등)
├───tailwind.config.ts      # Tailwind CSS 설정
└───tsconfig.json           # TypeScript 설정
```

### 2.1. 주요 폴더 설명

*   **`src/app/`**: Next.js 13+ App Router의 핵심입니다. 페이지 라우트, API 라우트, 레이아웃, 에러 핸들링 등이 이곳에서 정의됩니다.
    *   **`src/app/api/`**: 백엔드 API 엔드포인트 역할을 합니다. 클라이언트로부터 요청을 받아 `src/lib/services`의 로직을 호출하고 응답을 반환합니다.
    *   **`src/app/(pages)/`**: 사용자에게 보이는 UI 페이지들입니다. 각 폴더는 하나의 라우트를 나타냅니다 (예: `/monitor`, `/admin`). (참고: 실제 폴더명은 `(pages)`가 아닌 `admin`, `defects` 등 개별 페이지 이름입니다.)
*   **`src/components/`**: 애플리케이션 전반에서 재사용되는 UI 컴포넌트들을 모아둔 곳입니다. `app` 폴더 내의 페이지 컴포넌트에서 이들을 가져와 사용합니다.
*   **`src/lib/`**: 애플리케이션의 핵심 비즈니스 로직, 유틸리티 함수, 타입 정의, 전역 상태 관리 로직이 위치하는 곳입니다.
    *   **`src/lib/services/`**: DB, PLC 통신, 메인 모니터링 로직 등 외부 시스템과의 연동 및 핵심 비즈니스 로직을 캡슐화한 서비스 모듈들입니다. 각 서비스는 싱글톤 패턴으로 구현되어 애플리케이션 전역에서 단일 인스턴스를 사용합니다.
    *   **`src/lib/store/`**: Zustand를 이용한 전역 상태 관리 스토어들입니다. 세션, 알림, 모니터링 상태 등 클라이언트 상태를 효율적으로 관리합니다.
*   **`settings.json`**: 애플리케이션의 전역 설정 파일입니다. DB 연결 정보, PLC 연결 정보, Mock 모드 활성화 여부, 폴링 주기 등 변경될 수 있는 설정 값들을 여기에 저장하여 코드 수정 없이 시스템을 유연하게 운영할 수 있도록 합니다.
*   **`defect_rules.json`**: 불량 규칙 데이터를 저장하는 파일입니다. `src/lib/services/db.ts` 서비스에 의해 로드되고 관리됩니다.

## 3. 핵심 기능 분석

### 3.1. 인증 및 사용자 관리

본 시스템의 인증 및 사용자 관리는 다음과 같은 방식으로 이루어집니다. 현재는 간단한 하드코딩된 사용자 정보 기반으로 동작하며, 세션 관리를 위해 Zustand 스토어를 활용합니다.

#### 3.1.1. 주요 파일

*   **`src/lib/auth.ts`**: 인증 로직의 핵심 유틸리티 함수들을 정의합니다. 사용자 인증 (`authenticate`), 세션 검증 (`validateSession`), 로그아웃 (`logout`) 기능을 제공합니다.
*   **`src/lib/store/session-store.ts`**: Zustand를 사용하여 클라이언트와 서버에서 공유되는 세션 정보를 관리합니다. `createSession`, `getSession`, `deleteSession` 등의 함수를 통해 세션 상태를 조작합니다.
*   **`src/app/api/auth/login/route.ts`**: 로그인 요청(`POST /api/auth/login`)을 처리하는 API 라우트입니다. 사용자 이름과 비밀번호를 받아 `auth.ts`의 `authenticate` 함수를 호출하고, 성공 시 세션 ID를 쿠키에 설정하여 반환합니다.
*   **`src/app/api/auth/logout/route.ts`**: 로그아웃 요청(`POST /api/auth/logout`)을 처리하는 API 라우트입니다. 쿠키에 저장된 세션 ID를 통해 세션을 무효화합니다.
*   **`src/app/api/auth/me/route.ts`**: 현재 로그인된 사용자 정보(`GET /api/auth/me`)를 반환하는 API 라우트입니다. 세션 ID를 통해 사용자 정보를 검증하고 반환합니다.
*   **`src/app/login/page.tsx`**: 로그인 UI를 제공하는 페이지입니다.
*   **`src/components/LoginForm.tsx`**: 로그인 폼 컴포넌트입니다.
*   **`src/components/ProtectedRoute.tsx`**: 라우트 보호를 위한 컴포넌트입니다. 로그인되지 않은 사용자가 보호된 페이지에 접근하려 할 경우 로그인 페이지로 리다이렉트합니다.

#### 3.1.2. 동작 흐름

1.  **로그인 요청:**
    *   사용자가 `src/app/login/page.tsx` 또는 `src/components/LoginForm.tsx`를 통해 사용자 이름과 비밀번호를 입력하고 로그인합니다.
    *   클라이언트(`LoginForm.tsx`)는 `POST /api/auth/login` API 라우트로 요청을 보냅니다.
2.  **서버 측 인증:**
    *   `src/app/api/auth/login/route.ts`는 요청을 받아 사용자 이름과 비밀번호를 추출합니다.
    *   `src/lib/auth.ts`의 `authenticate(username, password)` 함수를 호출하여 사용자 정보를 검증합니다.
    *   `authenticate` 함수 내부에서는 `src/lib/store/session-store.ts`의 `createSession`을 호출하여 새로운 세션을 생성하고 세션 ID를 반환합니다.
    *   인증에 성공하면 `login/route.ts`는 세션 ID를 HTTP Only 쿠키에 설정하고 성공 응답을 반환합니다.
3.  **세션 유지 및 검증:**
    *   클라이언트는 이후 모든 요청에 이 세션 ID 쿠키를 자동으로 포함합니다.
    *   `src/components/ProtectedRoute.tsx`와 같은 보호된 라우트나 다른 API 라우트에서 사용자 정보가 필요한 경우, `GET /api/auth/me` API를 호출하거나 서버 컴포넌트에서 직접 쿠키의 세션 ID를 읽어 `auth.ts`의 `validateSession(sessionId)` 함수를 호출하여 현재 세션이 유효한지 확인하고 사용자 정보를 가져옵니다.
    *   `validateSession` 함수는 `session-store.ts`의 `getSession`을 사용하여 메모리에 저장된 세션 정보를 확인합니다.
4.  **로그아웃 요청:**
    *   사용자가 로그아웃을 요청하면 클라이언트는 `POST /api/auth/logout` API 라우트로 요청을 보냅니다.
    *   `src/app/api/auth/logout/route.ts`는 요청을 받아 쿠키의 세션 ID를 추출합니다.
    *   `src/lib/auth.ts`의 `logout(sessionId)` 함수를 호출하고, 내부적으로 `session-store.ts`의 `deleteSession`을 호출하여 세션을 무효화합니다.
    *   쿠키에서 세션 ID를 제거하고 로그아웃 성공 응답을 반환합니다.

#### 3.1.3. 데이터 연동

*   **`src/lib/auth.ts`** ↔ **`src/lib/store/session-store.ts`**: `auth` 유틸리티는 `session-store`를 통해 실제 세션 생성, 조회, 삭제를 수행합니다.
*   **Next.js API Routes** ↔ **`src/lib/auth.ts`**: API 라우트들은 `auth` 유틸리티 함수를 직접 호출하여 인증 및 세션 관련 로직을 처리합니다.
*   **클라이언트 컴포넌트** ↔ **Next.js API Routes**: 클라이언트 컴포넌트들은 `axios` 등을 사용하여 API 라우트와 통신합니다.

**특징:**
*   **하드코딩된 사용자:** 현재 `src/lib/auth.ts`에 `admin` 계정이 하드코딩되어 있습니다. 실제 서비스에서는 데이터베이스 연동이 필요합니다.
*   **메모리 기반 세션:** `session-store.ts`는 메모리 내에서 세션을 관리하므로, 서버 재시작 시 세션 정보가 초기화됩니다. 이 또한 실제 서비스에서는 Redis 등 영속적인 세션 저장소로 대체되어야 합니다.
*   **HTTP Only 쿠키:** 세션 ID는 보안을 위해 HTTP Only 쿠키에 저장되어 클라이언트 측 스크립트 접근을 방지합니다.

### 3.2. 모니터링 서비스

모니터링 서비스는 본 시스템의 핵심 로직으로, 생산 라인의 불량 상태를 감지하고 필요시 라인을 정지시키는 역할을 합니다. `src/lib/services/monitor.ts` 파일에 싱글톤 패턴으로 구현되어 애플리케이션 전역에서 단일 인스턴스로 동작합니다.

#### 3.2.1. 주요 파일

*   **`src/lib/services/monitor.ts`**: 모니터링 서비스의 핵심 로직을 담당합니다. 주기적인 불량 데이터 폴링, 불량 규칙 기반 라인 정지 판단, PLC 제어 명령 전송, 시스템 상태 관리 등을 수행합니다.
*   **`src/lib/services/db.ts`**: 불량 규칙 및 불량 데이터를 조회하는 데 사용됩니다. `monitor.ts`에서 `db.getRules()`와 `db.getAllDefectsAsync()` (또는 `db.checkLineStopProcedure()`)를 호출합니다.
*   **`src/lib/services/plc.ts`**: 라인 정지 및 재가동 명령을 실제 PLC에 전송하는 데 사용됩니다. `monitor.ts`에서 `plc.stopLine()`과 `plc.resetLine()`을 호출합니다.
*   **`src/lib/store/notification-store.ts`**: 라인 정지, 서비스 시작/정지 등 중요한 이벤트 발생 시 사용자에게 알림을 생성하는 데 사용됩니다.
*   **`settings.json`**: 폴링 주기(`polling.interval`), Mock 모드 활성화 여부(`mock.db`, `mock.plc`) 등 모니터링 서비스의 동작에 필요한 설정들을 정의합니다.
*   **`src/app/api/status/route.ts`**: 모니터링 서비스의 현재 상태(`GET /api/status`)를 클라이언트에게 제공하는 API 라우트입니다.
*   **`src/app/monitor/page.tsx`**: 실시간 라인 상태, 불량 카운트 등을 시각적으로 보여주는 모니터링 대시보드 페이지입니다.

#### 3.2.2. 동작 흐름

1.  **서비스 시작 (`monitorService.start()`):**
    *   초기화 시 또는 관리자 요청에 의해 `monitorService.start()`가 호출됩니다.
    *   `settings.json`에서 설정된 폴링 주기(`pollingInterval`)를 로드합니다.
    *   기존에 실행 중이던 인터벌 및 예약된 타이머를 모두 정리하여 중복 실행을 방지합니다.
    *   실제 PLC 모드인 경우, 초기 PLC 상태(`RUNNING` 또는 `STOPPED`)를 읽어와 서비스의 내부 라인 상태(`internalLineStatus`)와 동기화합니다.
    *   `processCycle()` 메서드를 즉시 한 번 실행한 후, `setInterval`을 사용하여 설정된 `pollingInterval`마다 `processCycle()`을 반복적으로 호출하도록 예약합니다.
    *   서비스 시작 알림을 생성합니다.
2.  **모니터링 주기 (`monitorService.processCycle()`):**
    *   `pollingInterval`마다 실행되는 핵심 메서드입니다.
    *   **Mock 모드인 경우:**
        *   `db.fetchRecentDefects()`를 호출하여 Mock 불량 데이터를 확률적으로 생성합니다 (라인이 `RUNNING` 상태일 때만).
        *   `db.getRules()`를 통해 모든 활성화된 불량 규칙을 가져옵니다.
        *   모든 Mock 불량(`db.getAllDefectsAsync()`)을 조회하여 각 규칙별로 불량 건수를 계산하고, 불량 코드별 임계값 초과 여부를 판단합니다. 이 과정은 Oracle DB의 `SP_CHECK_LINE_STOP` 프로시저의 로직을 시뮬레이션합니다.
        *   만약 하나라도 임계값을 초과하는 규칙이 있다면(`shouldStop = true`), `plc.stopLine()`을 호출하여 라인 정지 명령을 시뮬레이션하고, 서비스의 내부 라인 상태를 `STOPPED`로 업데이트합니다. 이 때 `stopLine` 호출 알림을 생성합니다.
        *   임계값에 도달하여 라인 정지를 유발한 규칙의 불량에 대해 `DEFECT_RESOLVE_DELAY` (기본 30초) 후 자동으로 해당 Mock 불량을 해소하고 라인을 재가동하는 타이머를 설정합니다 (`resolveMockDefectsByCode`, `resolveStop` 호출).
        *   모든 규칙이 임계값 미만으로 유지되면(`shouldStop = false`), 이전에 정지 상태였던 경우 `resolveStop()`을 호출하여 라인 재가동 명령을 시뮬레이션하고 서비스의 내부 라인 상태를 `RUNNING`으로 업데이트합니다.
    *   **실제 DB/PLC 모드인 경우:**
        *   `db.getRules()`를 통해 모든 활성화된 불량 규칙을 가져옵니다.
        *   각 활성화된 규칙에 대해 `db.checkLineStopProcedure(rule.code, rule.threshold)`를 호출합니다. 이 프로시저는 Oracle DB 내에서 현재 시간 기준 1시간 이내의 미해결 불량을 조회하고, 해당 규칙의 임계값 초과 여부를 판단하여 `STOP` 또는 `PASS` 결과를 반환합니다.
        *   만약 프로시저 결과가 `STOP`인 규칙이 하나라도 있다면, `plc.stopLine()`을 호출하여 실제 PLC에 라인 정지 명령을 전송하고, 서비스의 내부 라인 상태를 `STOPPED`로 업데이트합니다. 이 때 `stopLine` 호출 알림을 생성합니다.
        *   모든 프로시저 결과가 `PASS`인 경우(`shouldStop = false`), 이전에 정지 상태였던 경우 `plc.resetLine()`을 호출하여 실제 PLC에 라인 재가동 명령을 전송하고 서비스의 내부 라인 상태를 `RUNNING`으로 업데이트합니다.
    *   `currentDefects`와 `currentCounts` 같은 내부 상태 변수들을 업데이트하여 `getStatus()` 호출 시 최신 정보를 제공할 수 있도록 합니다.
3.  **서비스 정지 (`monitorService.stop()`):**
    *   관리자 요청에 의해 `monitorService.stop()`이 호출됩니다.
    *   `isRunning` 플래그를 `false`로 설정하고, 현재 실행 중인 `setInterval`을 `clearInterval`로 취소합니다.
    *   `stopSequenceId`를 증가시켜 현재 실행 중인 `processCycle` 내부에서 비동기 작업(`db.getAllDefectsAsync()`, `db.checkLineStopProcedure()`) 완료 후 `isRunning`이 `false`임을 감지하여 추가적인 작업을 중단하도록 합니다.
    *   예약된 모든 불량 해소 타이머(`defectResolveTimers`)를 취소합니다.
    *   서비스 정지 알림을 생성합니다.
4.  **라인 재가동 (`monitorService.resolveStop(reason)`):**
    *   관리자 요청(강제 재가동) 또는 Mock 모드에서 불량 해소 타이머 만료 시 호출됩니다.
    *   현재 라인 상태가 `STOPPED`인 경우에만 `plc.resetLine()`을 호출하여 PLC에 라인 재가동 명령을 전송하고 내부 라인 상태를 `RUNNING`으로 업데이트합니다.
    *   라인 재가동 알림을 생성합니다.

#### 3.2.3. 데이터 연동

*   **`monitor.ts`** ↔ **`db.ts`**: `monitor.ts`는 `db.ts`를 통해 불량 규칙을 조회하고, 최신 불량 데이터를 가져오거나 Oracle DB의 `SP_CHECK_LINE_STOP` 프로시저를 호출하여 라인 정지 판단을 위임합니다.
*   **`monitor.ts`** ↔ **`plc.ts`**: `monitor.ts`는 라인 정지 조건 충족 시 `plc.stopLine()`을 호출하고, 라인 재가동 시 `plc.resetLine()`을 호출하여 PLC를 제어합니다. `plc.readStatus()`를 통해 PLC의 현재 상태를 읽어 오기도 합니다.
*   **`monitor.ts`** ↔ **`notification-store.ts`**: `monitor.ts`는 서비스의 주요 이벤트(시작, 정지, 라인 정지/재가동) 발생 시 `createNotification()`을 호출하여 사용자에게 알림을 보냅니다.
*   **`settings.json`** ↔ **`monitor.ts`**: `monitor.ts`는 `settings.json` 파일에서 폴링 주기 및 Mock 모드 관련 설정을 로드하여 동작에 반영합니다.
*   **Next.js API Routes (`/api/status`)** ↔ **`monitor.ts`**: `/api/status` 라우트는 `monitor.ts`의 `getStatus()` 메서드를 호출하여 모니터링 서비스의 현재 상태 정보를 클라이언트에게 JSON 형태로 제공합니다.
*   **클라이언트 (`src/app/monitor/page.tsx`)** ↔ **Next.js API Routes (`/api/status`)**: 클라이언트의 모니터링 페이지는 주기적으로 `/api/status` API를 호출하여 최신 라인 상태, 불량 카운트 등을 업데이트하여 사용자에게 보여줍니다.

**특징:**
*   **단일 책임 원칙:** `monitor.ts`는 모니터링 로직에만 집중하고, DB 및 PLC 통신은 각각 `db.ts`와 `plc.ts`에 위임하여 모듈성을 높였습니다.
*   **Mock 모드 지원:** `settings.json` 설정을 통해 실제 DB/PLC 없이도 개발 및 테스트 환경을 구축할 수 있어 개발 효율성을 높입니다.
*   **싱글톤 패턴:** `monitorService`, `db`, `plc` 인스턴스는 애플리케이션 전역에서 단일 인스턴스로 관리되어 일관된 상태를 유지합니다.
*   **비동기 처리:** DB 및 PLC 통신은 비동기 작업으로 처리하여 메인 스레드 블로킹을 방지하고 애플리케이션의 반응성을 유지합니다.
*   **타임아웃 설정:** DB 및 PLC 연결 시 타임아웃을 설정하여 응답 없는 연결 시도를 방지합니다.

### 3.3. 불량 이력 관리

불량 이력 관리는 시스템에 기록된 모든 불량 데이터를 조회하고 필터링하여 분석하는 기능을 제공합니다.

#### 3.3.1. 주요 파일

*   **`src/lib/services/db.ts`**: 불량 데이터를 Oracle DB 또는 Mock DB에서 조회하는 핵심 로직을 담당합니다. `getAllDefectsAsync()` 메서드를 통해 모든 불량 데이터를 가져옵니다.
*   **`src/app/api/defects/route.ts`**: 불량 이력 조회 요청(`GET /api/defects`)을 처리하는 API 라우트입니다. 클라이언트로부터 `startDate`, `endDate`, `code` 등의 필터링 조건을 받아 `db.ts`를 통해 데이터를 조회하고, 필터링 후 결과를 반환합니다.
*   **`src/app/defects/page.tsx`**: 불량 이력을 보여주는 UI 페이지입니다. 날짜 범위 선택, 불량 코드 필터링 등의 기능을 포함하며, 테이블 형태로 불량 데이터를 표시합니다.

#### 3.3.2. 동작 흐름

1.  **불량 이력 페이지 접근:**
    *   사용자가 `src/app/defects/page.tsx`에 접근합니다.
    *   페이지는 초기 로딩 시 또는 사용자의 필터링 조건 변경 시 `GET /api/defects` API를 호출합니다.
2.  **API 요청 처리 (`/api/defects`):**
    *   `src/app/api/defects/route.ts`는 요청의 쿼리 파라미터(`startDate`, `endDate`, `code`)를 파싱합니다.
    *   `db.isMockMode` 플래그를 확인하여 Mock 모드 또는 실제 DB 모드 중 하나로 동작합니다.
    *   **Mock 모드인 경우:** `db.mockDefects`에 저장된 메모리 내 불량 데이터를 반환합니다.
    *   **실제 DB 모드인 경우:** `db.getAllDefectsAsync()` 메서드를 호출하여 Oracle DB에서 모든 불량 데이터를 비동기적으로 조회합니다. 이때 `queryFromOracle()` 메서드 내부에서 `ICOM_RECIEVE_DATA_NG` 테이블을 조회하며, `ACTION_DATE` 기준으로 최근 1시간 이내의 미처리 불량 데이터를 가져옵니다. (단, `/api/defects`에서는 `getAllDefectsAsync`가 모든 불량을 가져온 후, 이 라우트에서 날짜 필터링을 다시 적용하여 요청된 전체 기간의 불량을 처리합니다.)
    *   조회된 불량 데이터에 대해 `date-fns` 유틸리티를 사용하여 `startDate` 및 `endDate` 필터링을 적용하고, `code` 파라미터가 있다면 해당 불량 코드로 추가 필터링을 수행합니다.
    *   필터링된 불량 데이터를 JSON 형식으로 클라이언트에 반환합니다.
3.  **UI 렌더링:**
    *   `src/app/defects/page.tsx`는 API로부터 받은 불량 데이터를 기반으로 테이블 등을 렌더링하여 사용자에게 불량 이력을 시각적으로 보여줍니다.
    *   사용자는 날짜 선택기나 불량 코드 입력 필드를 통해 원하는 조건으로 불량 이력을 검색할 수 있습니다.

#### 3.3.3. 데이터 연동

*   **`src/app/defects/page.tsx`** ↔ **Next.js API Routes (`/api/defects`)**: 불량 이력 페이지는 API 라우트를 호출하여 불량 데이터를 가져옵니다.
*   **Next.js API Routes (`/api/defects`)** ↔ **`src/lib/services/db.ts`**: API 라우트는 `db` 서비스의 `getAllDefectsAsync()` (또는 `mockDefects`)를 사용하여 불량 데이터를 조회합니다.
*   **`src/lib/services/db.ts`** ↔ **Oracle DB (ICOM_RECIEVE_DATA_NG 테이블)**: `db` 서비스는 Oracle DB에 직접 연결하여 불량 데이터를 쿼리합니다.
*   **`date-fns`**: API 라우트 내에서 날짜 필터링 로직에 사용됩니다.

**특징:**
*   **Server-Side Data Fetching:** Next.js API Routes를 통해 서버에서 직접 DB 데이터를 조회하고 클라이언트에 전달하므로, 데이터 보안 및 성능 면에서 이점을 가집니다.
*   **유연한 필터링:** 날짜 범위 및 불량 코드별 필터링을 지원하여 사용자가 원하는 데이터를 쉽게 찾을 수 있도록 합니다.
*   **Mock 모드 통합:** 개발 및 테스트 시 실제 DB 연결 없이도 불량 이력 기능을 확인할 수 있도록 Mock 데이터를 효율적으로 활용합니다.
*   **데이터 정렬:** `db.ts`에서 Oracle DB 쿼리 시 `ACTION_DATE DESC`로 정렬하여 최신 불량부터 보여주도록 처리합니다.

### 3.4. 관리자 제어

관리자 제어 기능은 모니터링 서비스의 시작/정지, 불량 규칙 관리, 그리고 PLC 강제 제어(정지/재가동)와 같은 시스템의 핵심 동작을 관리자가 직접 제어할 수 있도록 합니다.

#### 3.4.1. 주요 파일

*   **`src/app/api/admin/control/route.ts`**: 모니터링 서비스 시작/정지, PLC 강제 정지/재가동 요청(`POST /api/admin/control`)을 처리하는 API 라우트입니다.
*   **`src/app/api/admin/rules/route.ts`**: 불량 규칙 목록 조회 및 규칙 추가(`GET/POST /api/admin/rules`)를 처리하는 API 라우트입니다.
*   **`src/app/api/admin/rules/[code]/route.ts`**: 특정 불량 규칙 조회, 수정, 삭제(`GET/PUT/DELETE /api/admin/rules/[code]`)를 처리하는 API 라우트입니다.
*   **`src/lib/services/monitor.ts`**: 서비스 시작(`start()`), 정지(`stop()`) 기능을 제공합니다.
*   **`src/lib/services/plc.ts`**: PLC 강제 정지(`stopLine()`), 재가동(`resetLine()`) 기능을 제공합니다.
*   **`src/lib/services/db.ts`**: 불량 규칙의 CRUD(`saveRule()`, `deleteRule()`, `getRules()`, `getRule()`) 기능을 제공합니다.
*   **`src/app/admin/page.tsx`**: 관리자 페이지 UI를 제공하며, 서비스 제어 버튼, 불량 규칙 목록 및 관리 UI를 포함합니다.

#### 3.4.2. 동작 흐름

1.  **모니터링 서비스 제어:**
    *   관리자 페이지(`src/app/admin/page.tsx`)에서 "서비스 시작" 또는 "서비스 정지" 버튼을 클릭합니다.
    *   클라이언트는 `POST /api/admin/control` API 라우트로 `action: "start"` 또는 `action: "stop"`을 포함한 요청을 보냅니다.
    *   `src/app/api/admin/control/route.ts`는 요청을 받아 `monitorService.start()` 또는 `monitorService.stop()`을 호출합니다.
    *   `monitor.ts`는 서비스의 `isRunning` 상태를 변경하고, `setInterval`을 시작하거나 중지하여 주기적인 모니터링 `processCycle`을 제어합니다.
    *   서비스 상태 변경 시 `notification-store.ts`를 통해 알림을 생성합니다.
2.  **PLC 강제 제어:**
    *   관리자 페이지에서 "PLC 강제 정지" 또는 "PLC 강제 재가동" 버튼을 클릭합니다.
    *   클라이언트는 `POST /api/admin/control` API 라우트로 `action: "force_stop"` 또는 `action: "force_reset"`을 포함한 요청을 보냅니다.
    *   `src/app/api/admin/control/route.ts`는 요청을 받아 `plc.stopLine(reason)` 또는 `plc.resetLine()`을 호출합니다.
    *   `plc.ts`는 실제 PLC에 MC Protocol을 사용하여 정지 또는 재가동 명령을 전송합니다. Mock 모드에서는 내부 `isStopped` 상태를 변경합니다.
3.  **불량 규칙 관리 (CRUD):**
    *   관리자 페이지는 `GET /api/admin/rules`를 호출하여 현재 등록된 불량 규칙 목록을 가져옵니다.
    *   **규칙 추가:** 관리자가 새로운 규칙 정보를 입력하고 저장하면, 클라이언트는 `POST /api/admin/rules`로 규칙 데이터를 전송합니다. `admin/rules/route.ts`는 `db.saveRule(rule)`을 호출하여 규칙을 `defect_rules.json` 파일에 저장합니다.
    *   **규칙 수정:** 관리자가 특정 규칙을 선택하여 수정하면, 클라이언트는 `PUT /api/admin/rules/[code]`로 수정된 규칙 데이터를 전송합니다. `admin/rules/[code]/route.ts`는 `db.saveRule(rule)`을 호출하여 기존 규칙을 업데이트합니다.
    *   **규칙 삭제:** 관리자가 특정 규칙을 삭제하면, 클라이언트는 `DELETE /api/admin/rules/[code]`로 요청을 보냅니다. `admin/rules/[code]/route.ts`는 `db.deleteRule(code)`를 호출하여 `defect_rules.json`에서 규칙을 제거합니다.

#### 3.4.3. 데이터 연동

*   **`src/app/admin/page.tsx`** ↔ **Next.js API Routes (`/api/admin/control`, `/api/admin/rules`, `/api/admin/rules/[code]`)**: 관리자 페이지는 API 라우트들을 통해 모니터링 서비스, PLC, 불량 규칙에 대한 제어 및 관리 작업을 수행합니다.
*   **Next.js API Routes (`/api/admin/control`)** ↔ **`src/lib/services/monitor.ts`**: 서비스 시작/정지 명령을 `monitor.ts`로 전달합니다.
*   **Next.js API Routes (`/api/admin/control`)** ↔ **`src/lib/services/plc.ts`**: 강제 정지/재가동 명령을 `plc.ts`로 전달합니다.
*   **Next.js API Routes (`/api/admin/rules`, `/api/admin/rules/[code]`)** ↔ **`src/lib/services/db.ts`**: 불량 규칙의 CRUD 작업을 `db.ts`로 위임합니다. `db.ts`는 `defect_rules.json` 파일에 규칙을 영구 저장합니다.

**특징:**
*   **중앙 집중식 제어:** `admin/control/route.ts` API를 통해 모니터링 서비스와 PLC 제어를 한 곳에서 관리하여 일관성을 유지합니다.
*   **규칙 영속성:** 불량 규칙은 `defect_rules.json` 파일에 JSON 형태로 저장되어 애플리케이션 재시작 후에도 유지됩니다.
*   **인가 (Authorization):** 관리자 기능이므로, 인증된 사용자(관리자 역할)만 해당 API에 접근할 수 있도록 보안이 강화되어야 합니다 (현재는 `ProtectedRoute`를 통해 클라이언트 측에서 제한).
*   **유연한 규칙 관리:** 웹 UI를 통해 불량 규칙을 쉽게 추가, 수정, 삭제할 수 있어 시스템 유지보수 편의성을 높입니다.

### 3.5. UI/UX 구성

본 시스템의 사용자 인터페이스(UI) 및 사용자 경험(UX)은 Next.js, React, Tailwind CSS를 기반으로 구축되었습니다. 사용자 친화적이며 직관적인 인터페이스를 제공하여 생산 라인 모니터링 및 관리를 효율적으로 수행할 수 있도록 설계되었습니다.

#### 3.5.1. 주요 파일 및 기술

*   **`src/app/layout.tsx`**: 애플리케이션의 전역 레이아웃을 정의합니다. 모든 페이지에 공통적으로 적용되는 `<head>` 메타데이터, `<html>` 구조, `Navbar` 컴포넌트의 조건부 렌더링 등을 담당합니다. `usePathname` 훅을 사용하여 현재 경로에 따라 `Navbar`를 숨길지 여부를 결정합니다.
*   **`src/app/page.tsx`**: 메인 랜딩 페이지입니다. 시스템 소개, 주요 메뉴 링크, 빠른 시작 가이드, 시스템 아키텍처 개요 등을 시각적으로 매력적인 디자인으로 제공합니다. `Link` 컴포넌트와 `lucide-react` 아이콘을 활용합니다.
*   **`src/components/Navbar.tsx`**: 상단 고정 네비게이션 바 컴포넌트입니다. 로고, 프로젝트 이름, 알림 드롭다운, 테마 토글, 사용자 메뉴 등을 포함합니다. `next/link`를 사용하여 페이지 간 이동을 처리합니다.
*   **`src/components/NotificationDropdown.tsx`**: 알림 메시지를 표시하고 관리하는 드롭다운 컴포넌트입니다. `src/lib/store/notification-store.ts`와 연동하여 실시간 알림을 받아옵니다.
*   **`src/components/ThemeToggle.tsx`**: 다크 모드와 라이트 모드를 전환할 수 있는 UI 컴포넌트입니다. 사용자의 시각적 선호도를 반영합니다.
*   **`src/components/UserMenu.tsx`**: 로그인 상태에 따라 사용자 이름 또는 로그인 버튼을 표시하고, 드롭다운 메뉴를 통해 로그아웃 등의 기능을 제공합니다.
*   **`src/app/globals.css`**: Tailwind CSS의 `@tailwind` 지시어와 함께 전역적으로 적용되는 커스텀 CSS 스타일을 정의합니다.
*   **`tailwind.config.ts`**: Tailwind CSS의 커스터마이징 설정을 정의합니다. 색상, 폰트, 간격 등 디자인 시스템을 구성합니다.
*   **`clsx`, `tailwind-merge`**: 조건부 Tailwind CSS 클래스를 쉽게 조합하고 최적화하는 데 사용되는 유틸리티 라이브러리입니다.

#### 3.5.2. 디자인 원칙 및 특징

*   **반응형 디자인:** 다양한 디바이스(데스크톱, 태블릿, 모바일)에서 최적화된 레이아웃과 사용자 경험을 제공하기 위해 Tailwind CSS의 반응형 유틸리티 클래스를 적극 활용합니다.
*   **다크 모드 지원:** `ThemeToggle` 컴포넌트를 통해 사용자가 선호하는 테마(다크/라이트)를 선택할 수 있도록 하여 시각적 편의성을 높입니다.
*   **일관된 스타일:** Tailwind CSS를 통해 전역적인 디자인 시스템을 구축하고, 재사용 가능한 컴포넌트를 사용하여 애플리케이션 전반에 걸쳐 일관된 UI/UX를 유지합니다.
*   **직관적인 네비게이션:** `Navbar`와 메인 페이지의 링크들은 사용자가 시스템의 다양한 기능에 쉽게 접근할 수 있도록 명확하고 시각적인 단서를 제공합니다. `lucide-react` 아이콘은 시각적 이해도를 높입니다.
*   **시각적 피드백:** 버튼 호버 효과, 활성 메뉴 강조 등 사용자 동작에 대한 즉각적인 시각적 피드백을 제공하여 인터랙션의 질을 높입니다.
*   **Next.js `Link` 컴포넌트:** 클라이언트 측 라우팅을 최적화하여 페이지 전환 시 빠른 로딩과 부드러운 사용자 경험을 제공합니다.

#### 3.5.3. 데이터 연동

*   **UI 컴포넌트** ↔ **Next.js API Routes**: 대부분의 UI 컴포넌트들은 `axios` 등의 HTTP 클라이언트를 사용하여 필요한 데이터를 Next.js API Routes로부터 가져오거나 서버에 데이터를 전송합니다.
*   **UI 컴포넌트** ↔ **Zustand 스토어**: `NotificationDropdown`, `UserMenu`와 같은 컴포넌트들은 `src/lib/store`에 정의된 Zustand 스토어(예: `notification-store`, `session-store`)와 연동하여 전역 상태를 구독하고 업데이트합니다.
*   **`usePathname` (Next.js Navigation Hook)**: `layout.tsx`에서 현재 URL 경로를 기반으로 `Navbar` 컴포넌트의 렌더링 여부를 조건부로 결정하는 데 사용됩니다.

**특징:**
*   **컴포넌트 기반 개발:** React의 컴포넌트 기반 아키텍처를 따라 UI를 작고 재사용 가능한 단위로 분할하여 개발 효율성과 유지보수성을 높였습니다.
*   **클라이언트 컴포넌트 (`"use client"`):** 사용자 인터랙션이 필요한 컴포넌트들은 `"use client"` 지시어를 사용하여 클라이언트 측에서 렌더링 및 동작하도록 명시합니다.
*   **Server Components (기본):** `"use client"` 지시어가 없는 컴포넌트나 페이지는 Next.js의 서버 컴포넌트로 동작하여 초기 로딩 성능을 최적화합니다.

## 4. 데이터 흐름 및 연동 방식

본 시스템은 Next.js의 통합된 프론트엔드/백엔드 환경을 기반으로 유기적인 데이터 흐름을 가집니다. 클라이언트의 요청이 서버 API를 거쳐 핵심 서비스 로직에 도달하고, 그 결과가 다시 클라이언트에게 전달되는 과정을 주요 기능별로 자세히 설명합니다.

### 4.1. 전반적인 데이터 흐름

```mermaid
graph TD
    A[클라이언트/UI (브라우저)] -->|1. 사용자 액션/요청| B(Next.js API Routes)
    B -->|2. 로직 처리 요청| C{핵심 서비스 로직 (monitor, db, plc)}
    C -->|3. DB/PLC 통신| D[Oracle DB / Mitsubishi PLC]
    D -->|4. 응답| C
    C -->|5. 처리 결과 반환| B
    B -->|6. API 응답| A
    C -->|7. 알림 생성| E(Zustand Notification Store)
    E -->|8. UI 알림 업데이트| A
```

**설명:**

1.  **클라이언트/UI (브라우저)에서 사용자 액션 또는 주기적인 요청 발생:**
    *   사용자가 로그인 버튼 클릭, 관리자 기능 조작, 불량 이력 조회 등의 액션을 취합니다.
    *   모니터링 페이지와 같이 주기적으로 데이터를 갱신해야 하는 경우, 클라이언트에서 일정 주기로 API를 호출합니다.
2.  **Next.js API Routes (백엔드)로 요청 전달:**
    *   클라이언트의 요청은 `src/app/api` 폴더에 정의된 Next.js API 라우트로 전달됩니다 (예: `/api/auth/login`, `/api/admin/control`, `/api/defects`).
    *   API 라우트는 요청의 메서드(GET, POST 등)와 본문/쿼리 파라미터를 파싱합니다.
3.  **핵심 서비스 로직 (`monitor`, `db`, `plc`) 호출:**
    *   API 라우트는 요청의 종류에 따라 `src/lib/services`에 정의된 적절한 서비스(예: `monitorService`, `db`, `plc`)의 메서드를 호출하여 비즈니스 로직을 수행합니다.
    *   예를 들어, `/api/admin/control`은 `monitorService.start()`나 `plc.stopLine()`을 호출합니다.
4.  **DB 또는 PLC와의 통신:**
    *   호출된 서비스 로직은 필요에 따라 Oracle DB(`db.ts`) 또는 미쓰비시 PLC(`plc.ts`)와 통신합니다.
    *   `db.ts`는 Oracle DB에 SQL 쿼리를 실행하거나 PL/SQL 프로시저(`SP_CHECK_LINE_STOP`)를 호출합니다.
    *   `plc.ts`는 MC Protocol을 사용하여 PLC의 레지스터 값을 읽거나 씁니다.
5.  **DB/PLC로부터 응답 수신:**
    *   DB 또는 PLC로부터 작업 결과 또는 데이터를 서비스 로직으로 반환합니다.
6.  **핵심 서비스 로직에서 처리 결과 반환:**
    *   서비스 로직은 DB/PLC의 응답을 처리하고, 최종 결과를 API 라우트로 반환합니다.
7.  **API Routes에서 클라이언트에게 응답 전달:**
    *   API 라우트는 서비스 로직으로부터 받은 결과를 JSON 형태로 가공하여 클라이언트에게 HTTP 응답으로 보냅니다.
8.  **Zustand Notification Store를 통한 UI 알림 업데이트 (선택 사항):**
    *   핵심 서비스 로직(특히 `monitor.ts`)에서 중요한 이벤트(라인 정지, 서비스 시작/정지 등)가 발생하면 `src/lib/store/notification-store.ts`의 `createNotification` 함수를 호출하여 전역 알림 상태를 업데이트합니다.
    *   클라이언트 UI의 `NotificationDropdown` 컴포넌트는 이 스토어를 구독하여 알림 목록이 변경될 때마다 UI를 자동으로 갱신하고 사용자에게 알림을 표시합니다.

### 4.2. 주요 연동 포인트

*   **`src/app/layout.tsx`** ↔ **`src/components/Navbar.tsx`**: 루트 레이아웃은 `Navbar`를 포함하며, `usePathname` 훅을 통해 특정 경로(`monitor`, `login`)에서는 `Navbar`를 숨깁니다.
*   **`src/components/Navbar.tsx`** ↔ **`src/components/NotificationDropdown.tsx`** ↔ **`src/lib/store/notification-store.ts`**: `Navbar`는 `NotificationDropdown`을 포함하고, `NotificationDropdown`은 `notification-store`의 상태를 구독하여 알림을 표시합니다.
*   **`src/components/Navbar.tsx`** ↔ **`src/components/UserMenu.tsx`** ↔ **`src/lib/store/session-store.ts`**: `Navbar`는 `UserMenu`를 포함하고, `UserMenu`는 `session-store`의 로그인 상태를 표시하며 로그아웃 기능을 제공합니다.
*   **`src/app/api/auth/*`** ↔ **`src/lib/auth.ts`** ↔ **`src/lib/store/session-store.ts`**: 인증 관련 API 라우트들은 `auth.ts` 유틸리티를 사용하고, `auth.ts`는 `session-store`를 통해 세션 상태를 조작합니다.
*   **`src/app/api/admin/control/route.ts`** ↔ **`src/lib/services/monitor.ts`**, **`src/lib/services/plc.ts`**: 관리자 서비스 제어 API는 `monitor.ts`의 서비스 시작/정지 및 `plc.ts`의 강제 정지/재가동 메서드를 호출합니다.
*   **`src/app/api/admin/rules/*`** ↔ **`src/lib/services/db.ts`**: 관리자 불량 규칙 API는 `db.ts`의 규칙 CRUD 메서드를 호출하여 `defect_rules.json`을 관리합니다.
*   **`src/app/api/defects/route.ts`** ↔ **`src/lib/services/db.ts`**: 불량 이력 API는 `db.ts`의 불량 조회 메서드를 호출합니다.
*   **`src/lib/services/monitor.ts`** ↔ **`src/lib/services/db.ts`** ↔ **Oracle DB**: 모니터링 서비스는 `db.ts`를 통해 Oracle DB의 불량 데이터 및 프로시저를 활용합니다.
*   **`src/lib/services/monitor.ts`** ↔ **`src/lib/services/plc.ts`** ↔ **Mitsubishi PLC**: 모니터링 서비스는 `plc.ts`를 통해 미쓰비시 PLC에 라인 제어 명령을 전송합니다.
*   **`settings.json`** ↔ **`src/lib/services/monitor.ts`**, **`src/lib/services/db.ts`**, **`src/lib/services/plc.ts`**: `settings.json` 파일은 애플리케이션의 전반적인 동작 방식을 설정하며, 각 서비스 모듈은 초기화 시 이 파일을 읽어 해당 설정을 적용합니다 (예: Mock 모드, 폴링 주기, DB/PLC 연결 정보).

이러한 연동 구조는 각 모듈의 독립성을 유지하면서도 시스템 전체가 유기적으로 동작하도록 설계되었습니다. 각 계층은 자신의 역할을 명확히 수행하며, 서비스 계층에서 핵심 비즈니스 로직을 처리하고 API 계층이 이를 웹 인터페이스로 노출하는 방식으로 구현되어 있습니다.

## 5. 환경 설정 (`settings.json`)

`settings.json` 파일은 애플리케이션의 동작 방식을 제어하는 핵심 설정 파일입니다. 이 파일은 개발 및 프로덕션 환경 모두에서 유연한 시스템 운영을 가능하게 하며, 특히 DB 연결 정보, PLC 연결 정보, 모니터링 서비스의 동작 주기, Mock 모드 활성화 여부 등을 정의합니다.

#### 5.1. 파일 위치 및 로딩

*   **위치**: 프로젝트의 루트 디렉터리(`D:\Project\OhSung-LineStop\settings.json`)에 위치합니다.
*   **로딩**: `src/lib/services`의 `monitor.ts`, `db.ts`, `plc.ts` 등 주요 서비스 모듈은 각자의 `constructor`에서 `fs.readFileSync`를 사용하여 `settings.json` 파일을 읽고 `JSON.parse`로 파싱하여 설정을 로드합니다.

#### 5.2. 설정 구조 및 주요 항목

```json
{
  "db": {
    "host": "192.168.110.222",
    "port": 1521,
    "service": "OSCW",
    "user": "INFINITY21_PIMMES",
    "password": "your_db_password"
  },
  "plc": {
    "ip": "192.168.0.1",
    "port": 5000,
    "address": "M100"
  },
  "polling": {
    "interval": 30
  },
  "mock": {
    "db": true,
    "plc": true,
    "db_defect_probability": 0.3
  }
}
```

*   **`db` (데이터베이스 설정):**
    *   `host`: Oracle DB 서버의 IP 주소.
    *   `port`: Oracle DB 서버의 포트 번호 (기본값: 1521).
    *   `service`: Oracle DB 서비스 이름 (SID 또는 서비스명).
    *   `user`: DB 접속 사용자 이름.
    *   `password`: DB 접속 비밀번호. **보안을 위해 실제 비밀번호는 환경 변수 등으로 관리하는 것이 권장됩니다.**
*   **`plc` (PLC 설정):**
    *   `ip`: 미쓰비시 PLC의 IP 주소.
    *   `port`: PLC 통신 포트 번호 (기본값: 5000).
    *   `address`: PLC의 제어 및 상태 확인에 사용되는 비트 주소 (예: `M100`). 이 주소의 값이 `1`이면 라인 정지, `0`이면 라인 가동으로 간주합니다.
*   **`polling` (모니터링 폴링 설정):**
    *   `interval`: `monitor.ts` 서비스가 DB로부터 불량 데이터를 조회하는 주기 (단위: 초).
*   **`mock` (Mock 모드 설정):**
    *   `db`: `db.ts` 서비스의 Mock 모드 활성화 여부 (true/false). `true`로 설정 시 실제 DB 대신 메모리 내 Mock 데이터를 사용합니다.
    *   `plc`: `plc.ts` 서비스의 Mock 모드 활성화 여부 (true/false). `true`로 설정 시 실제 PLC 대신 내부 메모리 상태를 사용합니다.
    *   `db_defect_probability`: `db.ts`의 Mock 모드에서 불량이 발생할 확률 (0.0 ~ 1.0). (예: `0.3`은 30% 확률로 불량 생성)

#### 5.3. 설정 변경의 영향

`settings.json` 파일의 값을 변경하면, 애플리케이션의 다음 동작에 영향을 미칩니다.

*   **DB/PLC 연결 정보:** 실제 DB나 PLC의 연결 정보가 변경되었을 때 이 파일을 수정하여 시스템이 올바른 장비와 통신하도록 합니다.
*   **Mock 모드 전환:** 개발 및 테스트 시 `mock.db`와 `mock.plc`를 `true`로 설정하여 실제 장비 없이도 애플리케이션의 기능을 테스트할 수 있습니다. 프로덕션 환경에서는 `false`로 설정해야 합니다.
*   **폴링 주기 조정:** `polling.interval` 값을 변경하여 불량 감지 주기를 시스템 운영 환경에 맞게 조절할 수 있습니다.

**주의사항:**
*   `settings.json` 파일은 애플리케이션이 실행되는 환경에 직접 배치되어야 합니다.
*   **비밀번호와 같은 민감한 정보는 `settings.json`에 직접 하드코딩하기보다, `.env` 파일과 같은 환경 변수 관리 시스템을 통해 주입하는 것이 보안상 매우 강력하게 권장됩니다.** 현재는 파일에 직접 포함되어 있으나, 배포 시에는 반드시 개선되어야 할 부분입니다.
*   `fs` 모듈을 사용하여 파일을 읽기 때문에, `settings.json` 변경 후에는 애플리케이션을 재시작해야 변경된 설정이 적용됩니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/YouHyuksoo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/YouHyuksoo)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
