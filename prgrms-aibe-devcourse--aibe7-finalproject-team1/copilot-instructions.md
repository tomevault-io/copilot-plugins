## aibe7-finalproject-team1

> 초기 MVP는 노트북을 대상으로 한다.

# AGENTS.md

## Project

전자제품 관리 및 중고거래 서비스.
초기 MVP는 노트북을 대상으로 한다.

## Tech Stack

- Java
- Spring Boot 4
- Gradle
- Spring Web MVC
- Spring Data JPA
- PostgreSQL
- Flyway
- Redis
- Spring Security + JWT
- Docker Compose
- JUnit 5
- Mockito
- Spotless

## Architecture

기본 구조:

Controller → Service → Repository → Entity

- Controller에 비즈니스 로직을 작성하지 않는다.
- Controller에서 Repository를 직접 호출하지 않는다.
- API Request / Response는 DTO를 사용한다.
- Entity를 API 응답으로 직접 반환하지 않는다.

## JPA

- 기본적으로 Lazy Loading을 사용한다.
- 불필요한 EAGER Loading을 사용하지 않는다.
- N+1 발생 가능성을 확인한다.
- 필요한 경우 Fetch Join, EntityGraph 등을 사용한다.
- Transaction 범위를 명확하게 관리한다.

## Database

- DB 스키마 변경은 반드시 Flyway Migration으로 관리한다.
- 기존 Migration 파일을 임의로 수정하지 않는다.
- 새로운 변경은 새로운 Migration 파일로 추가한다.
- `ddl-auto=update`에 의존하지 않는다.

## Security

- 인증이 필요한 API의 인증 / 인가 여부를 확인한다.
- 다른 사용자의 데이터에 접근할 수 없는지 확인한다.
- Password, Token, Secret, API Key를 코드나 로그에 노출하지 않는다.
- Refresh Token은 Redis에서 관리한다.

## Testing

새로운 기능이나 주요 비즈니스 로직에는 테스트를 작성한다.

작업 완료 후 가능한 경우 다음을 실행한다.

```bash
./gradlew spotlessCheck
./gradlew test
./gradlew build
```

실행하지 않은 테스트를 실행했다고 말하지 않는다.

Development Rules
작업 전에 관련 기존 코드를 먼저 확인한다.
기존 프로젝트 구조와 스타일을 우선한다.
요청하지 않은 대규모 리팩터링을 하지 않는다.
관련 없는 파일을 수정하지 않는다.
새로운 의존성 추가 전에 기존 의존성으로 해결 가능한지 확인한다.
과도한 추상화나 불필요한 디자인 패턴을 추가하지 않는다.
Git
main 브랜치에 직접 Push하지 않는다.
DB 변경 시 Flyway Migration을 함께 작성한다.
사용자가 요청하지 않은 Merge, Force Push, Branch 삭제를 수행하지 않는다.
Final Response

작업 완료 후 간단히 정리한다.

변경한 내용
변경한 주요 파일
테스트 결과
추가 확인이 필요한 사항

---
> Source: [prgrms-aibe-devcourse/AIBE7_FinalProject_Team1](https://github.com/prgrms-aibe-devcourse/AIBE7_FinalProject_Team1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
