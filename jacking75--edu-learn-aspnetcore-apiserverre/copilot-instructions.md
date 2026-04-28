## edu-learn-aspnetcore-apiserverre

> LLM과 함께 학습하며 ASP.NET Core 기반 API 게임 서버를 설계/구현하는 것이 목표.

# ASP.NET Core Web API 게임 서버 학습 저장소

## 목적
LLM과 함께 학습하며 ASP.NET Core 기반 API 게임 서버를 설계/구현하는 것이 목표.
학습 자료 + 참고 프로젝트 + 예제 코드가 포함된 교육용 저장소.

## 기술 스택
- **언어/프레임워크**: C# / ASP.NET Core (.NET 10.0)
- **DB**: MySQL (MySqlConnector + SQLKata 쿼리빌더, EF Core 미사용)
- **캐시**: Redis (CloudStructures 라이브러리)
- **로깅**: ZLogger (구조화 JSON 로깅) 또는 Serilog
- **모니터링**: Prometheus + Grafana
- **서버 OS**: Linux (Docker)
- **기타**: HttpClientFactory, Polly(재시도/서킷브레이커), FluentValidation, AspNetCoreRateLimit

## 디렉토리 구조
```
codes/                              # 예제 프로젝트 모음 (codes/README.md에 인덱스)
  GameAPIServer_Template/           # 단일 서버 API 템플릿 (net10.0, 71파일)
  MultiAPIServer_Template/          # 멀티 서버 구성 Game+Hive+Match (net10.0)
  practice_omok_game-1/             # 오목 게임 v1 (net10.0, 롱폴링)
  practice_omok_game-2/             # 오목 게임 v2 (net10.0)
  api_server_training_tany_farm/    # 타니 팜 (과제 명세만, FakeHiveServer만 존재)
  zlogger_test/                     # ZLogger 테스트 (net10.0)
docs/                               # 학습 문서
  guides/                           # 실습 가이드 (HttpClientFactory, DB트랜잭션, 통합테스트 등)
  patterns/                         # 설계 패턴 (Cache-Aside, DI)
  references/                       # 참고 자료 (디렉토리 구조, SqlKata, Serilog, ASP.NET Core 팁)
redis/                              # Redis 학습 통합
  docs/                             # CloudStructures 튜토리얼 6장 + 실습 과제 16개
  RedisExampleServer/               # 예제 서버 (net10.0, 6개 컨트롤러: String/List/Set/SortedSet/Lock/RateLimit)
ZLogger/                            # ZLogger 사용법
prometheus_grafana/                  # 모니터링 설정 및 튜토리얼
MetricsAPI/                         # 메트릭 API 예제
```

## 프로젝트 내 디렉토리 컨벤션
```
Controllers/    # API 컨트롤러
DTOs/           # 요청/응답 DTO
Services/       # 비즈니스 로직
  Interfaces/
Repository/     # DB 접근 계층
  Interfaces/
Middleware/      # 미들웨어 (버전 체크, 인증)
Models/         # DB 엔티티 모델
```

## 프로젝트 공통 파일명 규칙
- API 테스트: `apiTest.http`
- DB 스키마 문서: `DB_Schema.md`
- 에러 코드: `ErrorCode.cs` (UInt16 enum)

## 코딩 규칙 (coding_rule.md)
- dotnet/runtime 코딩 스타일 준수
- 클래스/구조체/함수/상수: PascalCase
- 매개변수: camelCase
- private/internal 필드: `_camelCase`, static 필드: `s_`, thread static: `t_`
- `private` 접근 제한자는 생략 (기본 적용)
- Allman 스타일 중괄호, 인덴트 4칸
- 가독성 > 간결성, 헝가리안 표기법 금지

## 서버 아키텍처 패턴
- **인증**: FakeHiveServer(외부 인증 서버 모의) → GameAPIServer
- **멀티 서버**: HiveAPIServer(인증) + GameAPIServer(게임) + MatchAPIServer(매칭)
- **캐싱**: Cache-Aside 패턴 (Redis) → docs/patterns/Cache-Aside_pattern.md
- **DB 트랜잭션**: docs/guides/how_to_db_transaction.md
- **환경 설정**: appsettings.{Environment}.json으로 개발/운영 분리
- **미들웨어 체인**: VersionCheck → CheckUserAuthAndLoadUserData → Controller

## 학습 순서 (권장)
1. C# 기초 → ASP.NET Core 기본 (미들웨어, DI, 라우팅)
2. DB 연동 (MySQL + SQLKata) → Redis (CloudStructures)
3. 로깅 (ZLogger/Serilog) → 환경 설정
4. GameAPIServer_Template 분석 → 자체 API 서버 구현
5. 모니터링 (Prometheus/Grafana) → 고급 주제 (gRPC, RateLimit 등)

## 빌드 및 실행
```bash
dotnet build
dotnet run
```
각 프로젝트별 .sln 파일 기준으로 빌드. `codes/` 하위 디렉토리에 개별 솔루션 존재.

### appsettings.json 비밀번호 관련 주의
각 프로젝트의 `appsettings.json`에는 MySQL/Redis 접속 비밀번호가 **평문으로 하드코딩**되어 있다.
이것은 **교육 목적의 더미 값**이며, 실제 운영 환경에서는 절대 사용하면 안 된다.

운영 환경에서의 비밀번호 관리 방법:
- **환경 변수**: `ConnectionStrings__GameDb` 등으로 설정
- **User Secrets** (`dotnet user-secrets`): 개발 환경 전용
- **Azure Key Vault / AWS Secrets Manager**: 클라우드 운영 환경
- **.env 파일 + Docker**: Docker Compose 환경에서 `env_file` 사용

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jacking75) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
