## pekko-kotlin-demo

> 이 파일은 Claude Code가 프로젝트를 이해하는 데 도움을 주는 문서입니다.

# CLAUDE.md

이 파일은 Claude Code가 프로젝트를 이해하는 데 도움을 주는 문서입니다.

## 프로젝트 개요

Apache Pekko Actor Model을 Kotlin으로 학습하기 위한 데모 프로젝트입니다. Pekko는 Akka 2.6.x의 Apache 라이선스 포크입니다.

## 기술 스택

| 기술 | 버전 | 용도 |
|-----|------|-----|
| Kotlin | 2.3.0 | 메인 언어 |
| Apache Pekko | 1.4.0 | Actor Model |
| Pekko HTTP | 1.3.0 | HTTP/WebSocket 서버 |
| Spring Boot | 4.0.1 | 웹 프레임워크 (일부 모듈) |
| JDK | 25 | Gradle Toolchain |
| Gradle | 9.2.1 | 빌드 도구 |
| Kotest | 6.0.5 | 테스트 프레임워크 |
| Exposed | 1.0.0-rc-4 | ORM (exposed 모듈) |

## 프로젝트 구조

```
pekko-kotlin-demo/
├── core/                    # Actor 기본, Streams, Ask 패턴
├── cluster/                 # 클러스터링, 싱글톤
├── persistence/             # 이벤트 소싱, 상태 복구
├── http/                    # REST API 서버 (port 8080)
├── grpc/                    # gRPC 서버 (port 50051)
├── exposed/                 # Exposed ORM + Actor 통합
├── websocket/               # Pure Pekko WebSocket
├── sse/                     # Pure Pekko SSE
├── websocket-cluster/       # WebSocket 클러스터
├── sse-cluster/             # SSE 클러스터
├── spring-boot/             # Spring Boot 기본 통합
├── spring-boot-cluster/     # Spring Boot + Pekko Cluster
├── spring-boot-optional-cluster/  # 선택적 클러스터 모드
├── spring-boot-websocket/   # Spring Boot WebSocket
├── spring-boot-sse/         # Spring Boot SSE
├── spring-boot-websocket-cluster/ # Spring Boot WS 클러스터
├── spring-boot-sse-cluster/ # Spring Boot SSE 클러스터
└── gradle/libs.versions.toml  # 의존성 버전 중앙 관리
```

## 주요 명령어

```bash
# 전체 빌드
./gradlew build

# 전체 테스트
./gradlew test

# 특정 모듈 실행
./gradlew :core:run
./gradlew :cluster:run
./gradlew :http:run
./gradlew :grpc:run

# Spring Boot 모듈 실행
./gradlew :spring-boot:bootRun
./gradlew :spring-boot-cluster:bootRun
```

## 코딩 컨벤션

### Actor 구현 스타일
- **클래스 기반 (AbstractBehavior)**: 복잡한 상태 관리에 적합
- **함수형 (Behaviors.receive)**: 간결한 상태 변경에 적합

### 테스트 스타일
- **Kotest DescribeSpec (BDD 스타일)** 사용
- `describe` - `context` - `it` 구조로 테스트 작성
- `shouldBe`, `shouldBeInstanceOf<T>()` 등 Kotest 매처 사용
- TestKit을 통한 Actor 테스트

```kotlin
// BDD 테스트 예시
class CounterActorTest : DescribeSpec({
    describe("CounterActor") {
        context("Increment 메시지를 받으면") {
            it("카운터를 증가시켜야 한다") {
                // 테스트 코드
            }
        }
    }
})
```

### 메시지 패턴
- **Tell**: Fire-and-Forget 방식 (`actorRef.tell(message)`)
- **Ask**: 요청-응답 패턴 (`AskPattern.ask()`)
- **Pipe**: 비동기 결과 전달 (`context.pipeToSelf()`)

## 의존성 관리

Gradle Version Catalog 사용 (`gradle/libs.versions.toml`):
```kotlin
// build.gradle.kts에서 사용
implementation(libs.pekko.actor.typed)
implementation(libs.bundles.pekko.core)
```

## 인코딩 설정

Windows 한글 인코딩 문제 해결을 위해 다음이 이미 설정됨:
- `gradle.properties`: JVM 인코딩 UTF-8
- `logback.xml`: 콘솔 인코딩 UTF-8
- `build.gradle.kts`: 실행 시 UTF-8 강제

## 주요 포트

| 모듈                    | 포트    |
|-----------------------|-------|
| http                  | 8080  |
| spring-boot           | 8081  |
| spring-boot-websocket | 8082  |
| spring-boot-sse       | 8083  |
| grpc                  | 50051 |

## 참고 문서

- [README.md](./README.md) - 프로젝트 소개 및 실행 방법
- [PEKKO_GUIDE.md](./PEKKO_GUIDE.md) - Actor Model 개념 및 Pekko 패턴 심화 학습
- 각 모듈별 README.md - 모듈별 상세 문서

---
> Source: [iceflower/pekko-kotlin-demo](https://github.com/iceflower/pekko-kotlin-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
