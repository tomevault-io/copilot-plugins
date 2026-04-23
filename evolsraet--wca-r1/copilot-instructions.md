## wca-r1

> - **프론트엔드/백엔드 개발자 동시 퇴사** (인수인계 없음)

# 위카옥션 v2 - Claude 개발 가이드

## 🚨 프로젝트 현황 (2024.07.02 기준)

### 상황 요약
- **프론트엔드/백엔드 개발자 동시 퇴사** (인수인계 없음)
- **문서화 거의 없음** - 코드 분석을 통한 기능 파악 필요
- **보안 취약점 다수 발견** - 즉시 수정 필요
- **핵심 기능은 작동** - API 레벨에서 회원가입/로그인/경매생성 확인됨

## 📋 프로젝트 정보

### 기술 스택
- **백엔드**: Laravel 10 + MySQL + Laravel Sanctum
- **프론트엔드**: Blade + Alpine.js (v2), Bootstrap 5.3 (sass) / Vue.js 3 (v1) 은 이전 버전으로 무시, 이전완성상태로 참고가능
- **개발환경**: Docker (Laravel Sail)
- **주요 패키지**: Spatie Permission, MediaLibrary, Socialite

### 주요 기능 모듈
1. **경매 시스템** - 차량 등록/입찰/낙찰 프로세스
2. **회원 관리** - 일반/딜러 회원가입, 소셜로그인
3. **결제 시스템** - 나이스페이먼츠 연동
4. **외부 API 연동** - 차량정보(나이스DNR), 시세(카머스), 탁송, SMS
5. **게시판** - 공지사항, 리뷰, 문의
6. **관리자** - 경매/회원/게시판 관리

## ⚠️ 즉시 수정 필요한 보안 이슈

### 🔴 심각도: 높음
1. **Mass Assignment 취약점** (`app/Models/User.php:45`)
   ```php
   // 현재: protected $guarded = [];
   // 수정: protected $fillable = ['name', 'email', 'phone', ...];
   ```

2. **API 키 노출** (`.env.example`)
   - 실제 운영 API 키들이 example 파일에 노출됨
   - 모든 키를 더미값으로 교체 필요

3. **CSRF 보호 우회** (`app/Http/Middleware/VerifyCsrfToken.php:30-32`)
   - 로컬환경에서 과도한 CSRF 예외 설정
   - 로그인/회원가입은 CSRF 보호 유지 필요

### 🟡 심각도: 중간
4. **하드코딩된 관리자 ID** (`app/Http/Controllers/Api/AuctionController.php:54`)

## 🏗️ 프로젝트 구조

### 디렉토리 구조
```
wca_v2/
├── app/
│   ├── Http/Controllers/    # API 및 웹 컨트롤러
│   ├── Models/             # Eloquent 모델
│   ├── Services/           # 비즈니스 로직 (AuctionService 등)
│   └── Jobs/               # 큐 작업 (알림, 상태변경 등)
├── routes/
│   ├── api.php            # API 라우트
│   ├── web.php            # 웹 라우트 (메인)
│   └── web.v2.php         # v2 웹 라우트
├── resources/
│   ├── v1/                # 구버전 (Vue.js) - 사용 중단
│   └── v2/                # 신버전 (Blade + Alpine.js) - 현재 사용
└── database/              # 마이그레이션, 시더
```

### 주요 API 엔드포인트
- `POST /api/users` - 회원가입
- `POST /login` - 로그인
- `GET|POST /api/auctions` - 경매 조회/생성
- `POST /api/bids` - 입찰
- `GET /api/payments` - 결제 정보

## 🚀 개발 환경 설정

### 필수 명령어 (Sail 사용)
```bash
# 환경 시작
./vendor/bin/sail up -d

# Laravel 상태 확인
./vendor/bin/sail artisan --version
./vendor/bin/sail artisan migrate:status

# 캐시 클리어
./vendor/bin/sail artisan config:clear
./vendor/bin/sail artisan route:clear
./vendor/bin/sail artisan view:clear

# 큐 워커 시작
./vendor/bin/sail artisan queue:work

# ERD 생성 (/laravel-erd)
./vendor/bin/sail artisan erd:generate


# 로그 확인
./vendor/bin/sail logs
```

### 테스트 명령어
```bash
# 단위 테스트
./vendor/bin/sail artisan test

# 라우트 목록 확인
./vendor/bin/sail artisan route:list
```

## 📝 개발 우선순위

### Phase 1: 보안 수정 (즉시 실행)
- [ ] Mass Assignment 취약점 수정
- [ ] API 키 노출 문제 해결
- [ ] CSRF 보호 설정 재검토
- [ ] 파일 접근 권한 검증 추가
- [ ] 하드코딩된 값들을 환경변수로 이동

### Phase 2: 코드 안정화 (1주일 내)
- [ ] 에러 로그 분석 및 버그 수정
- [ ] API 응답 표준화
- [ ] 유효성 검사 강화
- [ ] 예외 처리 개선
- [ ] 테스트 코드 작성

### Phase 3: 문서화 및 개선 (2주일 내)
- [ ] API 문서 작성
- [ ] 코드 주석 추가
- [ ] 배포 가이드 작성
- [ ] 성능 최적화
- [ ] 코드 리팩토링

### Phase 4: 기능 개선 (1개월 내)
- [ ] 사용자 경험 개선
- [ ] 새로운 기능 추가
- [ ] 모니터링 시스템 구축
- [ ] 백업 및 복구 시스템

## 🔧 주요 서비스 클래스

### AuctionService (`app/Services/AuctionService.php`)
경매 관련 핵심 비즈니스 로직 담당
- **상태 관리**: 경매 라이프사이클 관리 (진단→진행→낙찰→배송→완료)
- **비동기 처리**: Queue를 통한 알림 발송
- **외부 API**: 차량시세, 본인인증 등 연동
- **유효성 검사**: 상태 변경 시 엄격한 조건 검사

### UserService (`app/Services/UserService.php`)
회원 관리 및 인증 로직
- 회원가입/로그인 처리
- 소셜 로그인 연동
- 권한 관리

## 🚦 외부 API 연동 현황

### 연동 서비스
1. **나이스페이먼츠** - 결제 처리
2. **나이스DNR** - 차량정보 조회
3. **카머스(Carmerce)** - 차량시세 확인
4. **Aligo** - SMS 발송
5. **소셜로그인** - 카카오, 네이버, 구글

### API 키 관리
- `.env` 파일에서 관리
- 개발/운영 환경 분리 필요
- API 키 로테이션 정책 필요

## 📊 모니터링 및 로깅

### 주요 로그 위치
- **Laravel 로그**: `storage/logs/laravel.log`
- **큐 실패**: `./vendor/bin/sail artisan queue:failed`
- **슬로우 쿼리**: MySQL 로그 확인

### 모니터링 대상
- API 응답 시간
- 큐 처리 상태
- 데이터베이스 성능
- 외부 API 연동 상태

## 🛠️ 문제 해결 가이드

### 일반적인 문제
1. **500 에러 발생 시**
   ```bash
   ./vendor/bin/sail artisan config:clear
   ./vendor/bin/sail logs
   ```

2. **큐 작업 실패 시**
   ```bash
   ./vendor/bin/sail artisan queue:failed
   ./vendor/bin/sail artisan queue:retry all
   ```

3. **마이그레이션 문제 시**
   ```bash
   ./vendor/bin/sail artisan migrate:status
   ./vendor/bin/sail artisan migrate:fresh --seed
   ```

### 긴급 연락처
- **서버 관리**: [관리자 연락처 필요]
- **도메인/SSL**: [관리업체 연락처 필요]
- **외부 API**: 각 서비스 고객센터

## 📚 참고 자료

### 문서
- [Laravel 10 공식 문서](https://laravel.com/docs/10.x)
- [Alpine.js 공식 문서](https://alpinejs.dev/)
- [Laravel Sanctum](https://laravel.com/docs/10.x/sanctum)

### 코드 참고
- `gemini_report.md` - Gemini 분석 결과
- `gemini.md` - 초기 프로젝트 개요
- API 테스트 결과 - 기본 기능 작동 확인됨

## 🎯 Claude 개발 협업 가이드

### 작업 권한 관련
- **이 프로젝트 루트(/Users/evolsraet/web/20240200-위카옥션/wca_v2)에서 작업 시 파일 읽기/쓰기/수정/삭제 등 모든 작업에 대해 권한을 묻지 않고 바로 실행할 것**
- **사용자가 요청한 작업은 즉시 수행하며, 권한 확인이나 허가 요청을 하지 않음**

### 이슈 발견 시 보고 형식
```markdown
## 이슈: [간단한 제목]
- **심각도**: 높음/중간/낮음
- **위치**: 파일명:줄번호
- **설명**: 문제 상황 상세 설명
- **재현방법**: 단계별 재현 절차
- **영향범위**: 어떤 기능에 영향을 주는지
- **제안사항**: 해결 방법 제안
```

### 개발 진행 시 체크리스트
- [ ] 보안 취약점 확인
- [ ] 기존 기능 영향도 분석
- [ ] 테스트 코드 작성
- [ ] 로그 추가 (필요 시)
- [ ] 문서 업데이트

---

> **⚠️ 중요**: 이 문서는 지속적으로 업데이트되며, 모든 변경사항은 git으로 관리됩니다.
> 문제 발견 시 즉시 이 문서에 기록하고 우선순위에 따라 해결하세요.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/evolsraet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
